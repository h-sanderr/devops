# Proposta de práticas de DevOps e Git branching.

## Dicionário de termos

- _release_: lançamento.
- _feature_: recurso.

### Dicionário de Git

- `git clone <url-ou-uri-do-repositorio>`: clona repositório remoto com URL/URI \<url-ou-uri-do-repositorio\>.
- `git checkout <nome-branch>`: muda para branch com nome \<nome-branch\>.
- `git checkout -b <nova-branch>`: cria branch com nome \<nova-branch\> a partir da branch atual.
- `git add .`: adiciona todas as mudanças ao próximo _commit_.
- `git commit -m "exemplo"`: cria _commit_ com mensagem "exemplo".
- `git merge --no-ff <nome-branch>`: incorpora branch com nome \<nome-branch\> à branch atual, sem _fast forward_.
- `git push <nome-remoto> <nome-branch>`: sobe mudanças na branch com nome \<nome-branch\> ao remoto com nome \<nome-remoto\>.
- `git tag -a <x.y.z> -m "exemplo"`: cria _tag_ \<x.y.z\> a partir da branch atual, com mensagem "exemplo".
- `git branch -d <nome-branch>`: deleta branch com nome \<nome-branch\>.

## Motivação

- Atualmente, eu e o Renan utilizamos o git e versionamos o firmware de maneira diferente: por exemplo, eu trabalho com `main`, `develop` e uma branch para cada feature novo; o Renan trabalha apenas com `main` e `develop`. Isso causa confusão ao trabalhar junto no mesmo projeto, pois cada um manipula as branches de uma maneira.
- Falta de controle sobre _releases_ e versões dos firmwares. Especialmente quando próximo de entrar em ambiente de produção, pode causar problemas.
- Com o aumento do número de pessoas desenvolvendo firmware, ficará mais difícil trabalhar em grupo caso não haja padrões bem definidos de DevOps e Git branching.

## Proposta

A proposta é fazer o mais simples possível, levando em consideração que o desenvolvimento de firmware é diferente de outros tipos de software, já que o firmware pode depender do hardware e do ambiente de produção.

A figura abaixo mostra o modelo de Git branching sugerido. Ao longo deste documento, a branch `main` será chamada de `main` pois é o nome que o time já está acostumado a utilizar.

![Modelo de Git branching](./imgs/git_branching_model.png)

### Principais branches

As duas principais branches são `main` e `develop`. Ambas possuem um tempo de vida infinito, ou seja, nunca são deletadas.

- `main`: Possui apenas estados do firmware prontos para produção. **Importante:** toda vez que uma branch for incorporada à `main`, deve-se criar um _release_ no GitHub, com o hexadecimal ou binário do firmware, conforme descrito [aqui](#criando-releases-no-github). Assim, um desenvolvedor não depende do outro nem das ferramentas deste para gravar o firmware desejado.
- `develop`: Possui apenas estados do firmware com as últimas mudanças para o próximo _release_.

A figura abaixo ilustra a utilização das branches `main` e `develop`.

![Branches main e develop](./imgs/main_develop.png)

### Branches de suporte

As branches de suporte são as branches de _feature_, _release_ e _hotfix_. Estas branches auxiliam no desenvolvimento paralelo entre diversos membros da equipe, rastreamento fácil de _features_, e arrumar rapidamente problemas encontrados em produção.

#### Branches de _feature_

Branches de _feature_ são originadas da branch `develop` e são incorporadas à mesma. Qualquer nome pode ser utilizado para estas branches, exceto `main`, `develop`, `release-*` e `hotfix-*`.

São utilizadas para o desenvolvimento de novos _features_ para o próximo _release_ ou um _release_ futuro. Idealmente, cada _feature_ deve ter um _release_ alvo.

Quando incorporadas de volta à branch `develop`, são deletadas. Caso algo aconteça e for decidido que o _feature_ será descartado, basta deletar a branch sem incorporá-la de volta á branch `develop`.

Branches de _feature_ normalmente existem apenas localmente.

Exemplo de uso de uma branch de _feature_:

```console
$ git checkout develop # muda para branch develop
$ git checkout -b novo-feature develop # cria branch novo-feature a partir da develop

desenvolvimento completo do feature...

$ git add . # adiciona todas mudanças ao commit
$ git commit -m "desenvolvimento de novo feature" # faz commit das mudanças
$ git checkout develop # volta à develop
$ git merge --no-ff novo-feature # incorpora novo-feature à develop
$ git push origin develop # sobre mudanças na develop para repositório remoto
$ git branch -d novo-feature # deleta branch novo-feature
```

A flag `--no-ff` faz com que um novo _commit_ seja criado ao incorporar a nova branche, mantendo um histórico mais claro de commits. A figura abaixo ilustra a diferença entre uma incorporação utilizando a flag `--no-ff` (à esquerda) e uma com _fast forward_ (à direita).

#### Branches de _release_

Branches de _release_ são originadas da branch `develop` e incorporadas às branches `develop` e `main`.

A convenção de nomenclatura é `release-*`, em que `*` é a versão do `release`, de três números separados por um ponto (_e.g._, 1.2.0).

Branches de _release_ guardam os códigos de preparação para determinado _release_, permitindo correção de bugs menores e da versão do firmware. Desta forma, a branch `develop` também pode continuar recebendo novos _features_ para o próximo _release_.

O momento correto de criar uma branch de _release_ é quando todos os _features_ do _release_ já tiverem sido incorporados à branch `develop`, por isso a importância de saber o _release_ alvo de cada _feature_. É neste momento que a versão do firmware deve ser definida e as alterações associadas à versão devem ser realizadas.

Quando o estado da branch estiver realmente pronto para um _release_, ela deve ser incorporada às branches `main` e `develop`. Ao incorporar à branch `main`, deve-se criar uma _tag_ para futura referência a esta versão e criar um _release_ no GitHub, conforme descrito [aqui](#criando-releases-no-github).

Exemplo de uso de uma branch de _release_:

```console
$ git checkout develop
$ git checkout -b release-1.2.0 develop release-1.2.0 a partir da develop

preparação para o release...

$ git add .
$ git commit -m "preparação para release 1.2.0 finalizada"
$ git checkout main
$ git merge --no-ff release-1.2.0
$ git tag -a 1.2.0 -m "comentário sobre o release" # cria tag com base no estado atual do main
$ git push origin main
$ git checkout develop
$ git merge --no-ff release-1.2.0
$ git push origin develop
$ git push origin 1.2.0
$ git branch -d release-1.2.0
```

Se houver conflitos ao tentar incorporar à branch `develop`, o que é provável já que o número da versão foi alterado, deve-se resolver os conflitos e continuar a incorporação.

#### Branches de _hotfix_

Branches de _hotfix_ são originadas da `main` e são incorporadas de volta à `develop` e à `main`.

A convenção utilizada para nomear as branches de _hotfix_ é `hotfix-*`, em que `*` é o número relativo à _hotfix_, que é associado ao número da _release_ (_e.g_, se a versão da _release_ é 1.2 e for o primeiro _hotfix_, o nome da branche fica `hotfix-1.2.1`).

Branches de _hotfix_ não são planejadas e são criadas quando surge um problema numa versão já em produção que deve ser resolvido com urgência. Ao criar a branch de _hotfix_, a versão do firmware no código deve ser editada e, quando o problema for resolvido, a branch deve ser incorporada à `main` e à `develop`. Quando incorporada à `main`, deve-se criar uma _tag_ e criar um _release_ no GitHub, conforme descrito [aqui](#criando-releases-no-github). **Importante:** se uma branch de _release_ existir, em vez de incorporar a branch de _hotfix_ à `develop`, incorpora-se à branch de _release_. A figura abaixo ilustra o uso de branches de _hotfix_.

![Branches de hotfix](./imgs/hotfix_branches.png)

Exemplo de uso de branches de _hotfix_:

```console
$ git checkout -b hotfix-1.2.1 main

resolução do problema...

$ git commit -m "Fixed severe production problem"
$ git checkout main
$ git merge --no-ff hotfix-1.2.1
$ git tag -a 1.2.1 -m "comentário sobre o hotfix"
$ git push origin main
$ git checkout develop
$ git merge --no-ff hotfix-1.2.1
$ git push origin develop
$ git push origin 1.2.1
$ git branch -d hotfix-1.2.1
```

## Criando _releases_ no GitHub

[_Releases_ do GitHub](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository) são utilizadas para juntar em uma página só, códigos de determinada versão de uma aplicação com notas de lançamento, menções a contribuidores e outros arquivos associados. Esta é uma ferramenta útil pois permite a criação de uma página para cada versão de firmware com notas sobre a versão e, principalmente, arquivos binários ou hexadecimais do firmware, sem que haja necessidade de voltar a determinada versão e compilar novamente quando se deseja verificar ou testar algo. Por mais que não seja tão trabalhoso, pode haver problemas com a versão do SDK caso esta seja alterada, _overlays_ de placa, além da compilação depender de caminhos e configurações específicas dentro de determinados arquivos do projeto (por exemplo, os arquivos na pasta `.vscode/`, no caso de se utilizar as extensões para o VS Code dos SDKs da Nordic e da Espressif para compilar os projetos).

[Este link](https://docs.github.com/en/repositories/releasing-projects-on-github/managing-releases-in-a-repository#creating-a-release) ensina a criar _releases_ no GitHub e é só seguir o passo a passo e, com atenção aos seguintes pontos:

- No passo 1, utilizar um título no padrão `apelidodofirmware v[x.y.z]` (_e.g_, `xavierb 1.2.0`).
- No passo 4, usar uma _tag_ existente, que já deve ter sido subida para o GitHub.
- No passo 8, citar versão dos outros firmwares dependentes do que se está criando o _release_ (por exemplo, no caso do XavierB, citar versão do XavierL e XavierW).
- No passo 9, incluir os arquivos binários/hexadecimais do firmware, seguindo o padrão `apelidodofirmware_hardwarealvo_v[x.y.z].[bin|hex]`. Por exemplo, no caso do arquivo hexadecimal do XavierB, compilado para a placa LEW840F-M2-V3 da Fanstel, na versão 1.2.0, o arquivo fica `xavierb_lew840fm2v3_v1.2.0.hex`, tudo em minúsculo e sem traços. De preferência, deve-se incluir os arquivos binários/hexadecimais para todos os hardwares no qual foi testado, incluindo o nRF52840DK, por exemplo.

## Exemplo do fluxo de Git branching e DevOps proposto

1. Criar repositório no GitHub.
2. Criar a branch `develop` no GitHub.
3. Clonar repositório localmente:

   - Clonando com SSH:

     ```console
     $ git clone git@github.com:usuario-ou-organizacao/nome-do-repositorio.git
     ```

   - Clonando com HTTPS:

     ```console
     $ git clone https://github.com/usuario-ou-organizacao/nome-do-repositorio.git
     ```

4. Definir _features_ e associar a cada um deles um _release_ específico (opcional):

   | _Feature_          | _Release_ |
   | ------------------ | --------- |
   | Máquina de estados | 0.1.0     |
   | Controle dos LEDs  | 0.1.0     |
   | Leitura dos botões | 0.1.0     |
   | BLE scan           | 0.2.0     |
   | Comunicação UART   | 0.2.0     |
   | OTA                | 1.0.0     |

5. Trocar para a branch `develop`:

   ```console
   $ git checkout develop
   ```

6. Criar branch de _feature_ a partir da `develop`. Supondo que o primeiro _feature_ a ser adicionado seja a máquina de estados:

   ```console
   $ git checkout -b maquina-estados
   ```

7. Ao finalizar a máquina de estados, dar _commit_, incorporar à `develop`, subir mudanças e deletar `maquina-estados`:

   ```console
   $ git add .
   $ git commit -m "criação da máquina de estados"
   $ git checkout develop
   $ git merge --no-ff maquina-estados
   $ git push origin develop
   $ git branch -d maquina-estados
   ```

8. Repetir para os outros _features_ do _release_ 0.1.0.

9. Quando todos os _features_ do _release_ 0.1.0 estiverem prontos e incorporados à `develop`, criar branch de _release_ a partir da `develop`:

   ```console
   $ git checkout -b release-0.1.0
   ```

10. Realizar últimos ajustes para o _release_, incluindo mudar a versão do firmware dentro do código (0.1.0).

11. Quando tudo estiver pronto, dar _commit_, incorporar à `main`, criar _tag_ e subir mudanças na `main`:

    ```console
    $ git add .
    $ git commit -m "últimos ajustes para release 0.1.0"
    $ git checkout main
    $ git merge --no-ff release-0.1.0
    $ git tag -a 0.1.0 -m "primeiro release"
    $ git push origin main
    ```

12. Incorporar à `develop`, subir mudanças e deletar branch `release-0.1.0`:

    ```console
    $ git checkout develop
    $ git merge --no-ff release-0.1.0
    $ git push origin develop
    $ git branch -d release-0.1.0
    ```

13. Subir _tag_ 0.1.0 para o GitHub:

    ```console
    $ git push origin 0.1.0
    ```

14. Criar _release_ no GitHub conforme descrito [aqui](#criando-releases-no-github).

15. Supondo que encontrou-se um problema em produção que deve ser resolvido com urgência, criar branch de _hotfix_ a partir da `main`:

    ```console
    $ git checkout main
    $ git checkout -b hotfix-0.1.1
    ```

16. Após realizar os ajustes, dar _commit_, incorporar de volta à `main`, criar _tag_ e subir mudanças:

    ```console
    $ git add .
    $ git commit -m "solução de problema urgente"
    $ git checkout main
    $ git merge --no-ff hotfix-0.1.1
    $ git tag -a 0.1.1 "resolvido bug encontrado em produção"
    $ git push origin main
    ```

17. Incorporar também à `develop` , subir mudanças e deletar `hotfix-0.1.1`:

    ```console
    $ git checkout develop
    $ git merge --no-ff hotfix-0.1.1
    $ git push origin develop
    ```

18. Subir _tag_ 0.1.1 para o GitHub:

    ```console
    $ git push origin 0.1.1
    ```

19. Criar _release_ no GitHub conforme descrito [aqui](#criando-releases-no-github).

20. Repetir passos 6-19 para os próximos _releases_.

## Perguntas relevantes

### E o Jira?

Nada muda no Jira. Esta proposta se trata apenas de como as operações com Git são feitas.

### Já estamos no meio do desenvolvimento, o que fazer neste caso?

Sugestão: como nenhum _release_ foi feito, definir primeiro _release_ como 0.1.0 (quando for colocado no primeiro cliente) e começar a adotar as práticas de DevOps e Git branching sugeridas. Definir próximos _releases_ e os recursos a serem adicionados a cada um deles.

## Referências

- [How To Use Git-Flow In Embedded Software Development](https://medium.com/jumperiot/how-to-use-git-flow-in-embedded-software-development-dbb2a78da413)
- [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)
- [Git: diferenças entre fast forward e three way](https://www.lumis.com.br/a-lumis/blog/git.htm)
- [Git: Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
- [What Is a Git Merge Fast Forward?](https://blog.mergify.com/what-is-a-git-merge-fast-forward/)
- [Git Basics - Tagging](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
