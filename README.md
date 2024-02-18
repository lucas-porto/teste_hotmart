<p align="center">
  <a href="https://hotmart.com/pt-br">
    <img alt="Hotmart" src="https://upload.wikimedia.org/wikipedia/commons/a/a5/Logo_hotmart.png" width="240" />
  </a>
</p>
<h1 align="center">
    Avaliação de Cientista de Dados (AI Labs)
</h1>

<p align="center">
  <strong>
    Seu futuro cientista de dados, aqui!
  </strong>
</p>
<p align="justify">
O objetivo deste projeto é gerar a transcrição e tradução do vídeo original, além de um vídeo de saída de exemplo para comprovar o conceito de voice-over.

Então o que teremos em execução em nosso notebook (colocar link github) são todas as etapas do processo:

- Transcrição do vídeo
- Tradução do vídeo
- Criar um clipe e o voice-over (dublagem em inglês)
</p>

## :computer: Ambiente de execução

Este projeto foi criado em um notebook no Google Colab, utilizando sempre a versão de T4 GPU, para usuários que não são pagantes do Google Colab Pro, recomendo que executem os passos por partes, devido aos créditos serem limitados. Pode ocorrer de ser desconectado e somente conseguir executar o ambiente com GPU no dia seguinte.

Outra forma de execução é criar um ambiente virtual localmente, porém deve se atentar a ter uma GPU com no mínimo 14G para performance do Tortoise ser melhor aproveitada.

## :toolbox: Requisitos

- Wisper
  
Instalação do [Whisper](https://openai.com/research/whisper), que é um modelo Open Source da OpenAI como modelo de TTS (Text-to-Speech), iremos separar o áudio deste vídeo para podermos colocar como input do nosso modelo.


```
!pip install git+https://github.com/openai/whisper.git -q
```

- MoviePy

Instalação do [MoviePy](https://zulko.github.io/moviepy/), uma biblioteca para edição de vídeo.

```
!pip install moviepy
```

- SciPy

Instalação do [SciPy](https://scipy.org/), uma biblioteca para amplamente usada para trabalharmos com arrays N-dimensionais.

```
!pip3 install -U scipy
```

- Pydub

Instalação do [Pydub](https://pypi.org/project/pydub/), uma biblioteca para edição de áudio.

```
!pip3 install -U scipy
```

- Tortoise-TTS

Instalação do [Tortoise-TTS](https://docs.coqui.ai/en/latest/models/tortoise.html), um modelo de Text-to-Speech criado por James Betker, ele é mais lento que outros modelos TTS, mas apresenta um bom resultado no quesito de clonar uma voz. Ele usa a licença Apache 2.0 a qual devemos creditar o uso da ferramenta.

Neste ponto iremos clonar localmente e inicializar as bibliotecas necessárias para esta versão funcionar.

```
!git clone https://github.com/jnordberg/tortoise-tts.git
%cd tortoise-tts
!pip3 install -r requirements.txt
!pip3 install transformers==4.19.0 einops==0.5.0 rotary_embedding_torch==0.1.5 unidecode==1.3.5
!python3 setup.py install
```



## :hammer_and_pick: Processo de resolução

### Etapa 1: Criar Transcrição

A criação da transcrição do vídeo nós utilizamos o Whisper da OpenAi, ele é um modelo STT (Speech-to-Text), modelo voltado para reconhecimento de fala. Porém ele não se limita ao reconhecimento de fala e tem outras features como tradução da fala e identificação da língua.

Então primeiro separamos o áudio do vídeo usando a biblioteca MoviePy.

Após iremos criar a transcrição com o Whisper, foi criada a função <mark>setup_model</mark> para facilitar, então deve-se passar qual tarefa que deseja, neste passo é a <mark>"transcribe"</mark> para transcrição do áudio. Logo após salvando o resultado.

### Etapa 2: Criar Tradução

Para a tradução utilizamos o mesmo modelo e com a função que criamos, <mark>setup_model</mark>, passamos a tarefa <mark>"translate"</mark> para tradução para o inglês. 

Neste ponto poderíamos comparar a eficiência com outros modelos de NLP na execução da tradução, pode ser um teste para implementações futuras.


### Etapa 3: Criar Clipe com Voice-over

Esta etapa é a maior de todas do projeto e foi mais quebrada e não encapsulada para ser executada em partes caso haja desconexões.

Neste ponto teremos as seguintes etapas:

1. Criar o corte
2. Criar a tradução do corte
3. Clonar a voz do locutor do vídeo
    1. Gerar arquivos de treino
    2. Criar a pasta e salvar os arquivos
    3. Quebrar o nosso texto para se adequar ao tamanho que o modelo consegue criar áudio
    4. Gerar o áudio com a voz do locutor
    5. Ajustar o tempo do áudio
4. Retirar o áudio original do vídeo
5. Adicionar o novo áudio


#### Etapa 3.1: Criar o corte

Utiliza-se a biblioteca MoviePy, criei a função <mark>creating_clip</mark> para facilitar, sendo necessário passar os caminhos do vídeo de entrada, onde salvar o clipe e o tempo de início e fim do corte em segundos.


#### Etapa 3.2: Criar a tradução do corte

Mesma ação de criar a tradução do vídeo inicial, somente mudando o vídeo a ser traduzido na função <mark>setup_model</mark>.

#### Etapa 3.3: Clonar a voz do locutor do vídeo

A fim de não ter apenas as vozes que constumam ser utilizadas em vídeos dublados por AI, decidi criar a clonagem da voz do locutor e nesse passo utilizamos o Tortoise-TTS, que me deu uma voz melhor do que outros modelos com voz padrão e ligeiramente robótica.

##### Etapa 3.3.1: Gerar arquivos de treino

Para o treinamento no mínimo 2 áudios de 10 segundos geralmente são necessários, eu adicionei 30 áudios de 10 segundos para teste. 

Crio com base no áudio .mp3 criado do vídeo inteiro os clipes para treinamento.

##### Etapa 3.3.2: Criar a pasta e salvar os arquivos

É feito o setup para iniciar a utilização do modelo, nesse passo criamos a pasta local da voz custom a ser criada, selecionamos o preset de alta qualidade e fazemos a cópia dos áudios que estão no drive para o ambiente local do Colab onde fizemos o clone do git e criamos a pasta com a voz custom.


##### Etapa 3.3.3: Quebrar o nosso texto para se adequar ao tamanho que o modelo consegue criar áudio

Feito o setup, temos que usar uma função do próprio tortoise que auxilia na adequação das sentenças a serem criadas, já que não podemos criar textos muito grandes de uma só vez.


##### Etapa 3.3.4: Gerar o áudio com a voz do locutor

Para gerar o áudio, realizamos um loop para gerar trecho por trecho e no final unificar todas as partes com o áudio gerado.

##### Etapa 3.3.5: Ajustar o tempo do áudio

O áudio gerado ficou um pouco maior do que nosso vídeo, logo tivemos que manipular o áudio para caber nos 3 minutos que eram o tempo inicial. Isso pode trazer um pouco de mudança leve no tom dependendo do quanto seja alterado o nosso arquivo.

#### Etapa 3.4: Retirar o áudio original do vídeo

Este passo poderia ser executado anteriormente, mas é necessário salvarmos apenas o vídeo sem áudio a fim de inserirmos a nova camada de áudio. Criamos a função <mark>remove_audio_from_video</mark> para realizar essa retirada do áudio.

Também poderíamos tentar com ferramentas que pudesse manipular as faixas de áudio, assim mantendo o áudio de background. Serve para futuras implementações.


#### Etapa 3.4: Adicionar o novo áudio

Assim como o passo anterior utilizamos a biblioteca MoviePy e na função de auxílio <mark>add_audio_to_video</mark> criada executamos o passo de salvar o novo áudio ao nosso clipe.


## :heart: Obrigado

Foi um prazer realizar este projeto de avaliação, meu objetivo era gerar algo que fosse realizado somente em um ambiente sem mudança de ferramentas, então o áudio de background pode ser inserido de outras maneiras. 

Um ponto a melhorar na transcrição pode ser a melhoria da pontuação, algo que teria que ler mais como executar dentro do modelo do Whisper. Como o áudio teve que ser modificado no tempo, dá uma ligeira modificada na forma de falar e falta o ajuste do lipsync.

Obrigado por essa oportunidade de aprender e contem comigo, gostaria de ser parte da evolução neste ambiente de AI.