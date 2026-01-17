Gere um par de chaves privada/pública para autenticação sem senha.

No Linux, suas chaves são armazenadas em `~/.ssh`.

Se você já possui arquivos com `~/.ssh`os nomes `id_rsa`e `id_rsa.pub`, então você já possui um par de chaves. Anexe o conteúdo da sua chave pública (que é `id_rsa.pub`) ao arquivo ~/.ssh/authorized_keys do repositório Git.

```
$ scp ~/.ssh/id_rsa.pub user@git.repo:id_rsa.tmp
$ ssh user@git.repo
$ cat id_rsa.tmp >> .ssh/authorized_keys
```

Se você não tiver o par de chaves, gere um com

```
$ ssh-keygen -t rsa
```





# O guia definitivo de SSH – como configurar chaves SSH

[Tradutor: Daniel Rosa](https://www.freecodecamp.org/portuguese/news/author/daniel/)

[Autor: freeCodeCamp.org (em inglês)](https://www.freecodecamp.org/news/author/freeCodeCamp/)

![O guia definitivo de SSH – como configurar chaves SSH](https://www.freecodecamp.org/portuguese/news/content/images/size/w2000/2024/06/5f9c9ee3740569d1a4ca3fb6.jpg)

**Artigo original:** [The Ultimate Guide to SSH - Setting Up SSH Keys](https://www.freecodecamp.org/news/the-ultimate-guide-to-ssh-setting-up-ssh-keys/)

Boas-vindas ao guia definitivo de configuração de chaves SSH (do inglês, *Secure SHell*). Este tutorial mostrará para você o básico para a criação de chaves SSH e como gerenciar várias chaves e pares de chaves.

## **Criando um par de chaves SSH**

Abra o terminal e execute o seguinte comando:

```text
ssh-keygen
```

Você verá o texto a seguir:

```text
Generating public/private rsa key pair.
Enter file in which to save the key (/home/username/.ssh/id_rsa):
```

Pressione Enter para salvar suas chaves no diretório `/home/username/.ssh` padrão.

Em seguida, será solicitada a inserção de uma senha:

```text
Enter passphrase (empty for no passphrase):
```

Recomendamos inserir uma senha aqui para garantir uma camada a mais de segurança. Ao configurar uma senha, você pode evitar o acesso não autorizado aos seus servidores e contas caso alguém ponha as mãos em sua chave privada de SSH ou em sua máquina.

Depois de inserir sua senha e de confirmá-la, você verá o seguinte:

```text
Your identification has been saved in /home/username/.ssh/id_rsa.
Your public key has been saved in /home/username/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:/qRoWhRcIBTw0D4KpTUyK6YepyL6RQ2CQrtWsaicCb4 username@871e129f767b
The key's randomart image is:
+---[RSA 2048]----+
| .o=+....        |
|+.*o+o .         |
|+X.=o o          |
|@.=.oo .         |
|=O ...o S        |
|o.oo . .         |
|.E+ . . . .      |
|oo . ... +       |
|=.. .o. . .      |
+----[SHA256]-----+
```

Você, agora, tem um par de chaves, pública e privada, de SSH, que você pode usar para acessar servidores remotos e para lidar com a autenticação em programas de linha de comando, como o Git.

## **Gerenciando diversas chaves SSH**

Embora seja considerada uma boa prática ter apenas um único par de chaves pública e privada por dispositivos, há situações nas quais você necessita ter diversas chaves ou nas quais você tem nomes de chave não incomuns. Por exemplo, você pode estar usando um par de chaves SSH para trabalhar nos projetos internos de sua empresa, mas usará um par diferente de chaves para acessar os servidores de um cliente. Além disso, você pode usar um par diferente de chaves para acessar seu próprio servidor privado.

Gerenciar chaves SSH pode acabar sendo trabalhos quando você precisa usar uma segunda chave. Tradicionalmente, você usaria `ssh-add` para armazenar suas chaves no `ssh-agent`, digitando a senha para cada chave. O problema é que você precisaria fazer isso sempre que você reiniciasse seu computador, o que pode se tornar entediante bem rápido.

Uma solução melhor seria automatizar a adição de chaves e o armazenamento de senhas, especificando que chave usar ao acessar um servidor específico.

### **`config` no SSH**

É aí que aparece o `config` do SSH, que é um arquivo de configuração por usuário para a comunicação por meio do SSH. Crie um arquivo `~/.ssh/config` e abra-o para a edição:

```text
nano ~/.ssh/config
```

### **Gerenciando chaves de SSH com nomes personalizados**

A primeira coisa que vamos resolver ao usar esse arquivo `config` é evitar ter de adicionar chaves de SSH com nomes personalizado usando o `ssh-add`. Assumindo que sua chave de SSH privada seja chamada de `~/.ssh/id_rsa`, adicione o seguinte ao arquivo `config`:

```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
  IdentitiesOnly yes
```

Em seguida, certifique-se de que `~/.ssh/id_rsa` não esteja no `ssh-agent` abrindo outro terminal e executando o seguinte comando:

```text
ssh-add -D
```

Esse comando removerá todas as chaves da sessão do `ssh-agent` ativa neste momento.

Se, agora, você tentar fechar um repositório do GitHub, seu arquivo `config` usará a chave que está em `~/.ssh/ida_rsa`.

Aqui temos alguns exemplos úteis de configuração:

```bash
Host bitbucket-corporate
        HostName bitbucket.org
        User git
        IdentityFile ~/.ssh/id_rsa_corp
        IdentitiesOnly yes
```

Você, agora, pode usar `git clone git@bitbucket-corporate:company/project.git`

```bash
Host bitbucket-personal
        HostName bitbucket.org
        User git
        IdentityFile ~/.ssh/id_rsa_personal
        IdentitiesOnly yes
```

Também é possível usar `git clone git@bitbucket-personal:username/other-pi-project.git`

```text
Host myserver
        HostName ssh.username.com
        Port 1111
        IdentityFile ~/.ssh/id_rsa_personal
        IdentitiesOnly yes
        User username
        IdentitiesOnly yes
```

Por fim, você pode usar o SSH em seu servidor com `ssh myserver`. Já não é mais necessário inserir uma porta e um nome de usuário sempre que precisar do SSH para acessar seu servidor privado.

### **Gerenciamento de senhas**

A última dica tem a ver com o gerenciamento de senhas. Pode ser um tédio ter de inserir uma senha sempre que você inicia sua conexão com o SSH. Para contornar isso, podemos usar o software de gerenciamento de senhas que vem com o macOS e com várias distribuições do Linux.

Para este tutorial, usaremos o programa Keychain Access do macOS. Comece adicionando sua chave ao Keychain Access passando a opção `-K` ao comando `ssh-add`:

```bash
ssh-add -K ~/.ssh/id_rsa_whatever
```

Você, agora, pode ver sua chave de SSH no *Keychain Access.*

Se, no entanto, você remover as chaves do `ssh-agent` com `ssh-add -D` ou se reiniciar seu computador, será solicitada a sua senha novamente quando você tentar usar o SSH. Ocorre que ainda é necessário fazer mais uma configuração. Abra seu arquivo `config` do SSH com o comando `nano ~/.ssh/config` e adicione o seguinte:

```bash
Host *
  AddKeysToAgent yes
  UseKeychain yes
```

Com isso, sempre que você executar o `ssh`, ele buscará pelas chaves no *Keychain Access*. Se encontrar uma, sua senha não será mais solicitada. As chaves também serão adicionadas automaticamente ao `ssh-agent` sempre que você reiniciar sua máquina.

Agora que você sabe o básico sobre a criação de chaves SSH se sobre o gerenciamento de diversas chaves, fique à vontade para usar o `ssh` sempre que você quiser!