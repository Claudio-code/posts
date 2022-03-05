# Como colocar uma api Spring Boot com Docker



O objetivo é te ajudar a colocar uma api spring no docker, fazendo o build da api no contêiner com o **maven** e o **gradle** porque o modo de fazer o build muda dependendo do seu gerenciador de dependências favorito.

Nesse artigo eu não vou mostrar como criar uma api spring porque vou assumir que você sabe como funciona o básico do framework, por isso vou me concentrar em como colocar a api no docker e nas diferenças entre os gerenciadores de dependência.



## Criando o Dockerfile para fazer o build com o maven
