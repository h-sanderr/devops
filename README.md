# Proposta de práticas de DevOps e Git branching.

## Motivação

- Atualmente, eu e o Renan utilizamos o git de maneira diferente: eu trabalho com **main**, **develop** e uma branch para cada feature nova; o Renan trabalha apenas com **main** e **develop**. Isso causa confusão ao trabalhar junto no mesmo projeto, pois cada um manipula as branches de uma maneira.
- Falta de controle sobre releases e versões dos firmwares. Especialmente quando próximo de entrar em ambiente de produção, pode causar problemas.
- Com o aumento do número de pessoas desenvolvendo firmware, ficará mais difícil trabalhar em grupo caso não haja padrões bem definidos de DevOps e Git branching.

## Proposta

A proposta é fazer o mais simples possível, levando em consideração que o desenvolvimento de firmware é diferente de outros tipos de software, já que o firmware pode depender do hardware e do ambiente de produção.

## Referências

- [How To Use Git-Flow In Embedded Software Development](https://medium.com/jumperiot/how-to-use-git-flow-in-embedded-software-development-dbb2a78da413)
- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
