# Como colocar uma api Spring Boot com Docker

O objetivo é te ajudar a colocar uma api spring no docker, fazendo o build da api no contêiner com o **maven** e o **gradle** porque o modo de fazer o build muda dependendo do seu gerenciador de dependências favorito. A principal motivação a escrita desse post é que eu não encontrei esse conteudo de forma facil e objetiva.

Nesse artigo eu não vou mostrar como criar uma api spring porque vou assumir que você sabe como funciona o básico do framework, por isso vou me concentrar em como colocar a api no docker e nas diferenças entre os gerenciadores de dependência.

Nas duas formas não muda o modo de como a gente vai subir os containers com o docke-compose.

Uma coisa importante é que você nunca deve compilar um projeto seja de que linguagem for na sua máquina para rodar em outra plataforma em produção, 
porque por exemplo você estiver usando um computador com um processador arm e for rodar esse projeto na nuvem que provavelmente vai estar rodando algum x86 simplesmente o seu projeto não vai rodar
e mesmo que os dois forem da mesma plataforma muitas vezes por diferenças simples entre os hardwares o seu projeto pode não rodar por isso nesse artigo vamos compilar 
o projeto no Docker para garantir que o resultado seja o mesmo em todos os ambientes.  

## Criando o Dockerfile para fazer o build com o Maven

**1.** A primeira coisa que vamos fazer é construir a sequencia de comandos que vai fazer o build do nosso amado .jar e para isso vamos pegar uma imagem do **docker** com a versão do **maven** e do **jdk** 
que a gente precisa, no nosso caso nos vamos pegar a ultima lts do java e a ultima versão do maven e para isso vamos usar o seguinte comando e também vamos dar um apelido para essa parte:

```dockerfile
FROM maven:3-openjdk-17-slim as builder
```

Vamos agora passar para uma pasta onde será movido os arquivos do nosso projeto e vamos copiar os arquivos do nosso projeto.

```dockerfile
WORKDIR /builder
COPY . .
```

Agora vamos passar um comando para ele gerar o arquivo compilado da nossa api.

```dockerfile
RUN mvn clean package -DskipTests --batch-mode
```

**2.** Agora vamos pegar uma imagem mais leve do Java sem gerenciador de dependências e com uma distribuição de Linux mais leve vamos usar o Linux Alpine com o Java 17.

```dockerfile
FROM openjdk:17-jdk-alpine
```

Nesse momento a gente precisa copiar o nosso arquivo compilado .jar para dentro dessa imagem para isso vamos usar o apelido 
que nós definimos logo acima e passar o caminho de pastas até onde o Spring salvou o nosso arquivo.

```dockerfile
COPY --from=builder /builder/target/*.jar /app.jar
```

Por fim basta a gente chamar o nosso arquivo compilado como a gente faria rodando na máquina usando o comando ENTRYPOINT do Docker.

```dockerfile
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## Criando o Dockerfile para fazer o build com o Gradle

**1.** Nesse começo são os mesmos passos de quando a gente faz o build com o Maven vamos definir uma imagem com a versão do Gradle e do Java que queremos,
vamos ir para a pasta que queremos copiar os arquivos e vamos copiar o nosso projeto para dentro da imagem para poder compila-lo.

```dockerfile
## definindo a imagem e o apelido
FROM gradle:jdk17 as builder

## criando uma nova pasta e dando um cd para lá
WORKDIR /builder

## copiando os arquivos do projeto
COPY build.gradle .
COPY settings.gradle .
COPY src src
```

Agora vem a grande diferença o comando que gera o build que é bem simples.

```dockerfile
RUN gradle build --no-daemon
```

**2.** Com o Build Feito vamos selecionar outra imagem para rodar a api dessa vez vamos pegar uma versão slim do Debian que é bem leve também e novamente os passos se repetem.
Vamos copiar o .jar para dentro da imagem e rodar o comando para iniciar a api.

```dockerfile
## definindo a imagem que executara o nosso binario
FROM openjdk:17-slim-bullseye

## copiando o binario da outra imagem para essa
COPY --from=builder /builder/build/libs/*.jar /app.jar

## execultando a aplicação
ENTRYPOINT ["java", "-jar", "app.jar"]
```

##

## Agora alguns macetes para evitar dores de cabeça

#### Configurar caracteres especiais

Se você estiver usando um derivado do Cent Os adicione o comando:

```dockerfile
RUN echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

Se você estiver usando um derivado do Debian adicione o comando:

```dockerfile
RUN apt-get update \
    && apt-get upgrade -y \
    && apt-get install -y locales \
    && locale-gen en_US.UTF-8

RUN locale-gen en_US.UTF-8
ENV LANG en_US.UTF-8
```


#### Desabilitando o Daemon do Gradle

No comando de build do DockerFile do Gradle a gente passou uma flag ` --no-daemon ` que desabilita alguns processo que ficariam rodando em background e que não são necessários depois da compilação da nossa api.


#### Maven Flag para usar as propriedades de prod/dev

No Maven você pode ter dois arquivos de propriedades um com a config de prod e outro com a de dev ou teste, mais para isso você tem que criar os
arquivos no seguinte padrão `application-dev.properties`, `application-prod.properties`, `application-test.properties` ou qualquer outro nome que você queira para isso basta antes do ponto adicionar
o nome que representa essa configuração como nos exemplos dev, prod ou test.

O comando de build ficaria assim:

```dockerfile
RUN mvn clean package -Dspring-boot.run.profiles=prod -DskipTests --batch-mode
```


##### Links dos projetos usados para os exemplos de codigo

[Projeto usando o Maven](https://github.com/Claudio-code/spring-security)
[Projeto usando o Gradle](https://github.com/Claudio-code/geolocation-search-with-mongodb-spring)

Espero ter te ajudado, caso eu tenha errado em alguma coisa entre em contato comigo e me ajude melhorar.
