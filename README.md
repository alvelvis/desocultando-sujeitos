# Contando e desocultando sujeitos ocultos

O experimento visa identificar e desocultar sujeitos ocultos em três corpora em língua portuguesa: o DHBB, o OBras e o Bosque-UD.

Artigo sobre o experimento: (em breve).

## Passo a passo

Para reproduzir o experimento, disponibilizamos neste repositório grande parte dos códigos e arquivos necessários.

1) Por se tratar de corpora muito grandes, não disponibilizamos inteiros, mas relatamos o que fizemos para obtê-los a seguir:

O DHBB e o OBras foram obtidos do site da [Linguateca](http://linguateca.pt) no dia 26 de junho de 2020. Convertemos seu conteúdo, anotado no formato AC/DC, para texto cru utilizando o notebook (disponível no repositório) `convert-acdc-raw.ipynb`.

O resultado dessa conversão consta neste repositório:

- dhbb.zip
- obras.zip

Já o Bosque-UD foi obtido a partir do [GitHub](https://github.com/UniversalDependencies/UD_Portuguese-Bosque). Em 26 de junho de 2020, a versão do release era a 2.6, que está disponível neste repositório sob os nomes `bosque-ud-2.6-{train,dev,test}.conllu`. Concatenamos os três arquivos em um único com o seguinte comando:

```
$ cat bosque-ud-2.6-train.conllu bosque-ud-2.6-dev.conllu bosque-ud-2.6-test.conllu > bosque-ud-2.6.conllu
```

2) Treinamos um modelo no [UDPipe](http://ufal.mff.cuni.cz/udpipe) 1.2.0 com a partição de treino do Bosque-UD 2.6.

```
$ ./udpipe-1.2.0 --train --tokenizer --tagger --parser bosque-ud-2.6.udpipe bosque-ud-2.6-train.conllu
```

3) Anotamos tanto o texto do DHBB quanto o texto do OBras com o modelo treinado:

```
$ ./udpipe-1.2.0 --tokenize --tag --parse bosque-ud-2.6.udpipe dhbb.raw > dhbb.conllu
$ ./udpipe-1.2.0 --tokenize --tag --parse bosque-ud-2.6.udpipe obras.raw > obras.conllu
```

4) Rodamos o notebook `sujeito-oculto.ipynb` para realizar as buscas descritas no artigo e separar as frases cujos sujeitos podem ser desocultados (parte 1) e, de fato, desocultá-los (parte 2). Como resultado, ao final do notebook, geramos o corpus `bosque-ud-2.6-desocultado.conllu`.

5) Dividimos este corpus em três partições utilizando os seguintes comandos:

```
$ python3 split_conllu.py bosque-ud-2.6-desocultado.conllu
$ python3 generate_release.py .
$ mv pt-ud-train.conllu bosque-ud-2.6-desocultado-train.conllu
$ mv pt-ud-dev.conllu bosque-ud-2.6-desocultado-dev.conllu
$ mv pt-ud-test.conllu bosque-ud-2.6-desocultado-test.conllu
```

6) Treinamos um modelo com a partição de treino deste novo corpus:

```
$ ./udpipe-1.2.0 --train --tagger --parser bosque-ud-2.6-desocultado.udpipe bosque-ud-2.6-desocultado-train.conllu
```

7) Realizamos a validação dos dois modelos do Bosque-UD (desocultado e normal) removendo a anotação da partição de teste dos corpora correspondentes, mas sem destokenizá-los, pois a tokenização não está sendo avaliada neste experimento. O fizemos com os seguintes comandos:

```
$ python3 tokenizar_conllu.py bosque-ud-2.6-test.conllu bosque-ud-2.6-test-tokenizado.txt
$ python3 tokenizar_conllu.py bosque-ud-2.6-desocultado-test.conllu bosque-ud-2.6-desocultado-test-tokenizado.txt
$ python3 udpipe_vertical.py bosque-ud-2.6.udpipe bosque-ud-2.6-test-tokenizado.txt > bosque-ud-2.6-test-anotado.conllu
$ python3 udpipe_vertical.py bosque-ud-2.6-desocultado.udpipe bosque-ud-2.6-desocultado-test-tokenizado.txt > bosque-ud-2.6-desocultado-test-anotado.txt
```

8) Comparamos a qualidade da anotação com os seguintes comandos:

```
$ python3 conll18_ud_eval.py bosque-ud-2.6-test.conllu bosque-ud-2.6-test-anotado.conllu -v
$ python3 conll18_ud_eval.py bosque-ud-2.6-desocultado-test.conllu bosque-ud-2.6-desocultado-test-anotado.conllu -v
```
