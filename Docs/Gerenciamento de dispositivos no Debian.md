[toc]





# udev

## Nome

udev — gerenciamento dinâmico de dispositivos no Linux



## Descrição

O udev fornece ao software do sistema eventos de dispositivos, gerencia permissões de nós de dispositivos e pode criar links simbólicos adicionais no `/dev` diretório ou renomear interfaces de rede. O kernel geralmente apenas atribui nomes de dispositivos imprevisíveis com base na ordem de descoberta. Links simbólicos significativos ou nomes de dispositivos de rede oferecem uma maneira confiável de identificar dispositivos com base em suas propriedades ou configuração atual.

O daemon udev, udevd (8) , recebe eventos de dispositivo diretamente do kernel sempre que um dispositivo é adicionado ou removido do sistema, ou quando seu estado é alterado. Quando o udev recebe um evento de dispositivo, ele compara seu conjunto configurado de regras com vários atributos do dispositivo para identificá-lo. As regras correspondentes podem fornecer informações adicionais sobre o dispositivo para serem armazenadas no banco de dados udev ou para serem usadas na criação de nomes de links simbólicos significativos.

Todas as informações do dispositivo processadas pelo udev são armazenadas no banco de dados do udev e enviadas aos possíveis assinantes de eventos. O acesso a todos os dados armazenados e às fontes de eventos é fornecido pela biblioteca libudev.

## Configuração

Os arquivos de configuração do udev são colocados em `/etc/udev` e `/usr/lib/udev`. Todas as linhas em branco ou linhas que começam com '#' são ignoradas.



### Arquivo de configuração

O udev espera encontrar seu arquivo de configuração principal em `/etc/udev/udev.conf`. Ele consiste em um conjunto de variáveis que permitem ao usuário sobrescrever os valores padrão do udev. As seguintes variáveis podem ser definidas:

- `udev_root`

  Especifica onde colocar os nós de dispositivo no sistema de arquivos. O valor padrão é `/dev`.

- `udev_log`

  A prioridade de registro. Os valores válidos são as prioridades numéricas do syslog ou suas representações textuais: `err`, `info` e `debug`.

### Arquivos de regras

As regras do udev são lidas dos arquivos localizados no diretório de regras do sistema `/usr/lib/udev/rules.d`, no diretório de tempo de execução volátil `/run/udev/rules.d` e no diretório de administração local `/etc/udev/rules.d`. Todos os arquivos de regras são classificados e processados coletivamente em ordem lexicográfica, independentemente dos diretórios em que se encontram. No entanto, arquivos com nomes idênticos se substituem mutuamente. Os arquivos em `/etc/dev/regras` `/etc` têm a maior prioridade, e os arquivos em `/etc/dev/regras` `/run`têm precedência sobre arquivos com o mesmo nome em `/etc `/lib`/dev/regras`. Isso pode ser usado para substituir um arquivo de regras fornecido pelo sistema por um arquivo local, se necessário; um link simbólico em `/etc/dev/regras` `/etc`com o mesmo nome de um arquivo de regras em `/etc/dev/regras` `/lib`, apontando para `/etc/dev `/dev/null`/regras`, desativa completamente o arquivo de regras.

Os arquivos de regras devem ter a extensão .rw `.rules`; outras extensões serão ignoradas.

Cada linha no arquivo de regras contém pelo menos um par chave-valor. Existem dois tipos de chaves: de correspondência e de atribuição. Se todas as chaves de correspondência corresponderem ao seu valor, a regra é aplicada e as chaves de atribuição recebem o valor especificado.

Uma regra de correspondência pode renomear uma interface de rede, adicionar links simbólicos apontando para o nó do dispositivo ou executar um programa específico como parte do tratamento de eventos.

Uma regra consiste em uma lista de um ou mais pares chave-valor separados por vírgulas. Cada chave possui uma operação distinta, dependendo do operador utilizado. Os operadores válidos são:

- `==`

  Compare para verificar a igualdade.

- `!=`

  Compare para desigualdade.

- `=`

  Atribua um valor a uma chave. As chaves que representam uma lista são redefinidas e apenas esse valor único é atribuído.

- `+=`

  Adicione o valor a uma chave que contém uma lista de entradas.

- `:=`

  Atribua um valor a uma chave por fim; impeça quaisquer alterações posteriores.

Os seguintes nomes de chave podem ser usados para correspondência com propriedades de dispositivos. Algumas das chaves também correspondem a propriedades dos dispositivos pai no sysfs, não apenas ao dispositivo que gerou o evento. Se várias chaves que correspondem a um dispositivo pai forem especificadas em uma única regra, todas essas chaves deverão corresponder ao mesmo dispositivo pai.

- `ACTION`

  Combine o nome da ação do evento.

- `DEVPATH`

  Combine o caminho de desenvolvimento (devpath) do dispositivo de evento.

- `KERNEL`

  Associe o nome do dispositivo do evento.

- `NAME`

  Corresponde ao nome de uma interface de rede. Pode ser usado depois que a chave NAME tiver sido definida em uma das regras anteriores.

- `SYMLINK`

  Corresponde ao nome de um link simbólico que aponta para o nó. Pode ser usado depois que uma chave SYMLINK tiver sido definida em uma das regras anteriores. Pode haver vários links simbólicos; apenas um precisa corresponder.

- `SUBSYSTEM`

  Combine o subsistema do dispositivo de evento.

- `DRIVER`

  Atribua o nome do driver ao dispositivo do evento. Defina esta chave apenas para dispositivos que estejam vinculados a um driver no momento em que o evento for gerado.

- `ATTR{*`filename`*}`

  Corresponde aos valores de atributos do sistema de arquivos (sysfs) do dispositivo de evento. Espaços em branco à direita nos valores dos atributos são ignorados, a menos que o próprio valor de correspondência especificado contenha espaços em branco à direita.

- `KERNELS`

  Procure no caminho de desenvolvimento (devpath) acima por um nome de dispositivo correspondente.

- `SUBSYSTEMS`

  Procure no caminho de desenvolvimento (devpath) um nome de subsistema de dispositivo correspondente.

- `DRIVERS`

  Procure no caminho de desenvolvimento (devpath) acima por um nome de driver de dispositivo correspondente.

- `ATTRS{*`filename`*}`

  Procure no caminho de desenvolvimento (devpath) um dispositivo com valores de atributo sysfs correspondentes. Se várias `ATTRS`correspondências forem especificadas, todas elas devem corresponder ao mesmo dispositivo. Espaços em branco à direita nos valores dos atributos são ignorados, a menos que o próprio valor de correspondência especificado contenha espaços em branco à direita.

- `TAGS`

  Procure no caminho de desenvolvimento (devpath) um dispositivo com a tag correspondente.

- `ENV{*`key`*}`

  Comparar com o valor de uma propriedade do dispositivo.

- `TAG`

  Compare com a etiqueta do dispositivo.

- `TEST{*`octal mode mask`*}`

  Verifica a existência de um arquivo. Uma máscara de modo octal pode ser especificada, se necessário.

- `PROGRAM`

  Executa um programa para determinar se há uma correspondência; a chave é verdadeira se o programa retornar com sucesso. As propriedades do dispositivo são disponibilizadas ao programa em execução no ambiente. A saída padrão (stdout) do programa está disponível na chave RESULT.

- `RESULT`

  A chave corresponde à string retornada pela última chamada do PROGRAMA. Essa chave pode ser usada na mesma regra ou em qualquer regra posterior a uma chamada do PROGRAMA.

A maioria dos campos suporta correspondência de padrões no estilo shell. Os seguintes caracteres de padrão são suportados:

- `*`

  Corresponde a zero ou mais caracteres.

- `?`

  Corresponde a qualquer caractere individual.

- `[]`

  Corresponde a qualquer caractere individual especificado entre colchetes. Por exemplo, a string de padrão 'tty[SR]' corresponderia a 'ttyS' ou 'ttyR'. Intervalos também são suportados através do caractere '-'. Por exemplo, para corresponder ao intervalo de todos os dígitos, o padrão [0-9] poderia ser usado. Se o primeiro caractere após o '[' for um '!', quaisquer caracteres não entre colchetes serão correspondidos.

As seguintes chaves podem receber valores:

- `NAME`

  O nome a ser usado para uma interface de rede. O nome de um nó de dispositivo não pode ser alterado pelo udev, apenas links simbólicos adicionais podem ser criados.

- `SYMLINK`

  O nome de um link simbólico que aponta para o nó. Cada regra correspondente adiciona esse valor à lista de links simbólicos a serem criados. Vários links simbólicos podem ser especificados separando os nomes por um espaço. Caso vários dispositivos reivindiquem o mesmo nome, o link sempre apontará para o dispositivo com a maior prioridade de link (link_priority). Se o dispositivo atual for removido, os links serão reavaliados e o dispositivo com a próxima maior prioridade de link se tornará o proprietário do link. Se nenhuma prioridade de link for especificada, a ordem dos dispositivos (e qual deles é o proprietário do link) será indefinida. Além disso, os nomes dos links simbólicos nunca devem entrar em conflito com os nomes de nós de dispositivo padrão do kernel, pois isso resultaria em comportamento imprevisível.

- `OWNER, GROUP, MODE`

  As permissões para o nó do dispositivo. Cada valor especificado substitui o valor padrão compilado.

- `ATTR{*`key`*}`

  O valor que deve ser gravado em um atributo sysfs do dispositivo de evento.

- `ENV{*`key`*}`

  Defina o valor de uma propriedade do dispositivo. Os nomes de propriedades que começam com um ponto (.) não são armazenados no banco de dados nem exportados para eventos ou ferramentas externas (executadas, por exemplo, pela chave de correspondência PROGRAM).

- `TAG`

  Atribui uma etiqueta a um dispositivo. Isso é usado para filtrar eventos para usuários da funcionalidade de monitoramento da libudev ou para enumerar um grupo de dispositivos etiquetados. A implementação só funciona de forma eficiente se apenas algumas etiquetas estiverem atribuídas a um dispositivo. Ela se destina apenas a contextos com requisitos específicos de filtragem de dispositivos e não como um sinalizador de uso geral. O uso excessivo pode resultar em tratamento ineficiente de eventos.

- `RUN`

  Adicionar um programa à lista de programas a serem executados em um dispositivo específico.Se nenhum caminho absoluto for fornecido, espera-se que o programa esteja localizado em /usr/lib/udev; caso contrário, o caminho absoluto deve ser especificado. O nome do programa e os argumentos subsequentes são separados por espaços. Aspas simples podem ser usadas para especificar argumentos com espaços.Isso só pode ser usado para tarefas de curta duração. Executar um processo de evento por um longo período pode bloquear todos os eventos subsequentes para este dispositivo ou para um dispositivo dependente. Iniciar daemons ou outros processos de longa duração não é apropriado para o udev.

- `LABEL`

  Um rótulo nomeado para o qual um comando GOTO pode saltar.

- `GOTO`

  Salta para o próximo RÓTULO com um nome correspondente.

- `IMPORT{*`type`*}`

  Importe um conjunto de variáveis como propriedades do dispositivo, dependendo de *`type`*:`program`Executa um programa externo especificado como o valor atribuído e importa sua saída, que deve estar no formato de chave de ambiente. A especificação do caminho, a separação de comando/argumento e o uso de aspas funcionam como em `RUN`.`file`Importe um arquivo de texto especificado como o valor atribuído, cujo conteúdo deve estar no formato de chave de ambiente.`db`Importa uma única propriedade especificada como o valor atribuído do banco de dados do dispositivo atual. Isso só funciona se o banco de dados já estiver preenchido por um evento anterior.`cmdline`Importa uma única propriedade da linha de comando do kernel. Para sinalizadores simples, o valor da propriedade é definido como '1'.`parent`Importe as chaves armazenadas do dispositivo pai lendo a entrada do banco de dados do dispositivo pai. O valor atribuído `IMPORT{parent}`é usado como um filtro de nomes de chave para importação (com a mesma correspondência de padrões no estilo shell usada para comparações).

- `WAIT_FOR`

  Aguarda até que um arquivo fique disponível ou até que um tempo limite de 10 segundos expire. O caminho é relativo ao dispositivo sysfs; se nenhum caminho for especificado, aguarda até que um atributo apareça.

- `OPTIONS`

  Opções de regras e dispositivos:`link_priority=*`value`*`Especifique a prioridade dos links simbólicos criados. Dispositivos com prioridades mais altas sobrescrevem os links simbólicos existentes de outros dispositivos. O padrão é 0.`event_timeout=`Número de segundos que um evento aguarda a conclusão das operações antes de desistir e se encerrar.`string_escape=*`none|replace`*`Normalmente, caracteres de controle e outros caracteres potencialmente inseguros são substituídos em strings usadas para nomear dispositivos. O modo de substituição pode ser especificado com esta opção.`static_node=`Aplique as permissões especificadas nesta regra ao nó de dispositivo estático com o nome especificado. Nós de dispositivo estáticos podem ser fornecidos por módulos do kernel ou copiados de `/usr/lib/udev/devices`. Esses nós podem não ter um dispositivo de kernel correspondente no momento em que o udevd é iniciado; eles podem acionar o carregamento automático de módulos do kernel.`watch`Monitore o nó do dispositivo com o inotify; quando o nó é fechado após ter sido aberto para escrita, um evento de alteração (uevent) é sintetizado.`nowatch`Desative o monitoramento de um nó de dispositivo com o inotify.

Os campos `NAME`` <nome_do_campo>` `SYMLINK`, `PROGRAM``<nome_do_campo> `, `OWNER``<nome_do_campo> `, `<nome_do_campo> `GROUP`` `MODE` e `<nome_do_campo>` `RUN` suportam substituições simples de strings. As `RUN` substituições são realizadas após o processamento de todas as regras, imediatamente antes da execução do programa, permitindo o uso de propriedades do dispositivo definidas por regras de correspondência anteriores. Para todos os outros campos, as substituições são realizadas durante o processamento da regra individual. As substituições disponíveis são:

- `$kernel`,`%k`

  O nome do kernel para este dispositivo.

- `$number`,`%n`

  O número do kernel para este dispositivo. Por exemplo, 'sda3' tem o número de kernel '3'.

- `$devpath`,`%p`

  O caminho de desenvolvimento do dispositivo.

- `$id`,`%b`

  O nome do dispositivo correspondeu ao pesquisar o caminho de desenvolvimento para cima por `SUBSYSTEMS`, `KERNELS`, `DRIVERS`e `ATTRS`.

- `$driver`

  O nome do driver do dispositivo correspondeu durante a busca no caminho de desenvolvimento (devpath) para cima, em busca de `SUBSYSTEMS`, `KERNELS`, `DRIVERS`e `ATTRS`.

- `$attr{*`file`*}`,`%s{*`file`*}`

  O valor de um atributo sysfs encontrado no dispositivo onde todas as chaves da regra corresponderam. Se o dispositivo correspondente não possuir tal atributo e um teste anterior de KERNELS, SUBSYSTEMS, DRIVERS ou ATTRS tiver selecionado um dispositivo pai, o atributo desse dispositivo pai será utilizado.Se o atributo for um link simbólico, o último elemento do destino do link simbólico será retornado como valor.

- `$env{*`key`*}`,`%E{*`key`*}`

  Um valor de propriedade do dispositivo.

- `$major`,`%M`

  O número principal do kernel para o dispositivo.

- `$minor`,`%m`

  O número secundário do kernel para o dispositivo.

- `$result`,`%c`

  A string retornada pelo programa externo solicitado com PROGRAM. Uma única parte da string, separada por um espaço, pode ser selecionada especificando o número da parte como um atributo: `%c{N}`. Se o número for seguido pelo caractere '+', essa parte, juntamente com todas as partes restantes da string resultante, será substituída:`%c{N+}`

- `$parent`,`%P`

  O nome do nó do dispositivo pai.

- `$name`

  O nome atual do dispositivo. Se não for alterado por uma regra, será o nome do dispositivo do kernel.

- `$links`

  Uma lista dos links simbólicos atuais, separados por espaços. O valor só é definido durante um evento de remoção ou se uma regra anterior já tiver atribuído um valor.

- `$root`,`%r`

  O valor udev_root.

- `$sys`,`%S`

  O ponto de montagem do sysfs.

- `$devnode`,`%N`

  O nome do nó do dispositivo.

- `%%`

  O próprio caractere '%'.

- `$$`

  O próprio caractere '$'.

Autor

Escrito por Greg Kroah-Hartman`<``greg@kroah.com>` e Kay Sievers . Com a valiosa ajuda de Dan Stekloff e muitos outros.`<kay.sievers@vrfy.org>`

## Veja também

udevd (8) , udevadm (8)





# udevd

## Nome

udevd — daemon de gerenciamento de eventos

## Sinopse

`udevd` [ `--daemon`] [ `--debug`] [ `--children-max=`] [ `--exec-delay=`] [ `--resolve-names=early|late|never`] [ `--version`] [ `--help`]



## Descrição

O udevd escuta os uevents do kernel. Para cada evento, o udevd executa as instruções correspondentes especificadas nas regras udev. Veja udev (7) .

Na inicialização, o conteúdo do diretório `/usr/lib/udev/devices` é copiado para `/dev`. Se os módulos do kernel especificarem nós de dispositivo estáticos, esses nós serão criados mesmo sem um dispositivo de kernel correspondente, para permitir o carregamento sob demanda dos módulos do kernel. As permissões correspondentes especificadas nas regras udev são aplicadas a esses nós de dispositivo estáticos.

O comportamento do daemon em execução pode ser alterado com o **comando udevadm control** .

## Opções

- `--daemon`

  Desanexar e executar em segundo plano.

- `--debug`

  Imprime mensagens de depuração no stderr.

- `--children-max=`

  Limitar o número de eventos executados em paralelo.

- `--exec-delay=`

  Número de segundos para atrasar a execução das instruções RUN. Isso pode ser útil ao depurar falhas do sistema durante a inicialização a frio causadas pelo carregamento de módulos do kernel que não funcionam.

- `--resolve-names=`

  Especifique quando o udevd deve resolver os nomes de usuários e grupos. Quando definido como `early`(o padrão), os nomes serão resolvidos quando as regras forem analisadas. Quando definido como , `late`os nomes serão resolvidos para cada evento. Quando definido como , `never`os nomes nunca serão resolvidos e todos os dispositivos pertencerão ao root.

- `--version`

  Número da versão para impressão.

- `--help`

  Imprimir texto de ajuda.

## Ambiente

- `UDEV_LOG=`

  Defina a prioridade de registro.

## Linha de comando do kernel

- `udev.log-priority=`

  Defina a prioridade de registro.

- `udev.children-max=`

  Limitar o número de eventos executados em paralelo.

- `udev.exec-delay=`

  Número de segundos para atrasar a execução das instruções RUN. Isso pode ser útil ao depurar falhas do sistema durante a inicialização a frio causadas pelo carregamento de módulos do kernel que não funcionam.

Autor

Escrito por Kay Sievers`<``kay.sievers@vrfy.org>` .

## Veja também

udev (7) , udevadm (8)



# udevadm

## Nome7892840813505



udevadm — ferramenta de gerenciamento udev

## Sinopse

`udevadm` [ `--debug`] [ `--version`] [ `--help`]

`udevadm info *`options`*`

```
udevadm trigger [options]
udevadm settle [options]
```

`udevadm control *`command`*`

```
udevadm monitor [options]
```

`udevadm test [options] *`devpath`*`

`udevadm test-builtin [options] *`command`* *`devpath`*`



## Descrição

O udevadm espera um comando e opções específicas do comando. Ele controla o comportamento em tempo de execução do udev, solicita eventos do kernel, gerencia a fila de eventos e fornece mecanismos simples de depuração.

## OPÇÕES

- `--debug`

  Imprime mensagens de depuração no stderr.

- `--version`

  Número da versão para impressão.

- `--help`

  Imprimir texto de ajuda.



### informações udevadm*`options`*

Consulta o banco de dados udev para obter informações sobre dispositivos armazenadas nele. Também pode consultar as propriedades de um dispositivo a partir de sua representação no sistema de arquivos sysfs para auxiliar na criação de regras udev que correspondam a esse dispositivo.

- `--query=*`type`*`

  Consulta o banco de dados para obter dados de um tipo específico de dispositivo. É necessário o nome `--path`ou o link simbólico `--name`para identificar o dispositivo especificado. As consultas válidas são: **nome** , **link simbólico** , **caminho** , **propriedade** , **todas** .

- `--path=*`devpath`*`

  O caminho do dispositivo a ser consultado.

- `--name=*`file`*`

  O nome do nó do dispositivo ou um link simbólico para consulta.

- `--root`

  O diretório raiz do udev: `/dev`. Se usado em conjunto com uma consulta **de nome** ou **link simbólico** , a consulta retorna o caminho absoluto, incluindo o diretório raiz.

- `--run`

  O diretório de tempo de execução do udev: `/run/udev`.

- `--attribute-walk`

  Imprime todas as propriedades do sysfs do dispositivo especificado que podem ser usadas em regras udev para corresponder ao dispositivo especificado. Imprime todos os dispositivos ao longo da cadeia, até a raiz do sysfs, que podem ser usados em regras udev.

- `--export`

  Imprima os resultados como pares chave/valor. Os valores devem estar entre aspas simples.

- `--export-prefix=*`name`*`

  Adicione um prefixo ao nome da chave dos valores exportados.

- `--device-id-of-file=*`file`*`

  Imprima os números major e minoritário do dispositivo subjacente onde o arquivo está localizado.

- `--export-db`

  Exporte o conteúdo do banco de dados udev.

- `--cleanup-db`

  Limpe o banco de dados udev.

- `--version`

  Versão para impressão.

- `--help`

  Imprimir texto de ajuda.

### gatilho udevadm [ opções ]

Solicita eventos de dispositivo do kernel. Utilizado principalmente para reproduzir eventos no momento da conexão a frio do sistema.

- `--verbose`

  Imprima a lista de dispositivos que serão acionados.

- `--dry-run`

  Não acione o evento de fato.

- `--type=*`type`*`

  Acione um tipo específico de dispositivos. Os tipos válidos são: **dispositivos** , **subsistemas** . O valor padrão é **dispositivos** .

- `--action=*`action`*`

  Tipo de evento a ser acionado. O valor padrão é **alteração** .

- `--subsystem-match=*`subsystem`*`

  Acione eventos para dispositivos que pertencem a um subsistema correspondente. Esta opção pode ser especificada várias vezes e suporta correspondência de padrões no estilo do shell.

- `--subsystem-nomatch=*`subsystem`*`

  Não acione eventos para dispositivos que pertencem a um subsistema correspondente. Esta opção pode ser especificada várias vezes e suporta correspondência de padrões no estilo do shell.

- `--attr-match=*`attribute`*=*`value`*`

  Dispara eventos para dispositivos com um atributo sysfs correspondente. Se um valor for especificado junto com o nome do atributo, o conteúdo do atributo será comparado com o valor fornecido usando correspondência de padrões no estilo do shell. Se nenhum valor for especificado, a existência do atributo sysfs será verificada. Esta opção pode ser especificada várias vezes.

- `--attr-nomatch=*`attribute`*=*`value`*`

  Não acione eventos para dispositivos com um atributo sysfs correspondente. Se um valor for especificado junto com o nome do atributo, o conteúdo do atributo será comparado com o valor fornecido usando correspondência de padrões no estilo do shell. Se nenhum valor for especificado, a existência do atributo sysfs será verificada. Esta opção pode ser especificada várias vezes.

- `--property-match=*`property`*=*`value`*`

  Acione eventos para dispositivos com um valor de propriedade correspondente. Esta opção pode ser especificada várias vezes e suporta correspondência de padrões no estilo do shell.

- `--tag-match=*`property`*`

  Acione eventos para dispositivos com uma tag correspondente. Esta opção pode ser especificada várias vezes.

- `--sysname-match=*`name`*`

  Aciona eventos para dispositivos com um nome de dispositivo sys correspondente. Esta opção pode ser especificada várias vezes e suporta correspondência de padrões no estilo shell.

- `--parent-match=*`syspath`*`

  Acionar eventos para todos os dispositivos filhos de um determinado dispositivo.

### udevadm se estabelecer [ opções ]

Monitora a fila de eventos do udev e encerra a execução se todos os eventos atuais forem tratados.

- `--timeout=*`seconds`*`

  Número máximo de segundos que o sistema aguarda até que a fila de eventos fique vazia. O valor padrão é 120 segundos. Um valor de 0 verificará se a fila está vazia e retornará imediatamente.

- `--seq-start=*`seqnum`*`

  Aguarde apenas os eventos que ocorrerem após o número de sequência fornecido.

- `--seq-end=*`seqnum`*`

  Aguarde apenas os eventos anteriores ao número de sequência fornecido.

- `--exit-if-exists=*`file`*`

  Pare de esperar se o arquivo já existir.

- `--quiet`

  Não imprima nenhuma saída, como as entradas restantes na fila, quando o tempo limite for atingido.

- `--help`

  Imprimir texto de ajuda.

### controle udevadm*`command`*

Modifique o estado interno do daemon udev em execução.

- `--exit`

  Sinalize e aguarde a finalização do udevd.

- `--log-priority=*`value`*`

  Define o nível de log interno do udevd. Os valores válidos são as prioridades numéricas do syslog ou suas representações textuais: `err`, `info`e `debug`.

- `--stop-exec-queue`

  Sinalize ao udevd para parar de executar novos eventos. Os eventos recebidos serão enfileirados.

- `--start-exec-queue`

  Sinalize ao udevd para habilitar a execução de eventos.

- `--reload`

  Sinaliza ao udevd para recarregar os arquivos de regras e outros bancos de dados, como o índice de módulos do kernel. Recarregar regras e bancos de dados não aplica alterações a dispositivos já existentes; a nova configuração será aplicada apenas a novos eventos.

- `--property=*`KEY`*=*`value`*`

  Defina uma propriedade global para todos os eventos.

- `--children-max=`*`value`*

  Defina o número máximo de eventos que o udevd processará simultaneamente.

- `--timeout=`*`seconds`*

  O número máximo de segundos que o usuário deve aguardar por uma resposta do udevd.

- `--help`

  Imprimir texto de ajuda.

### monitor udevadm [ opções ]

Escuta os eventos uevent do kernel e os eventos enviados por uma regra udev e imprime o caminho do dispositivo (devpath) do evento no console. Pode ser usado para analisar o tempo de ocorrência do evento, comparando os carimbos de data/hora do evento uevent do kernel e do evento udev.

- `--kernel`

  Imprima os eventos do kernel.

- `--udev`

  Imprima o evento udev após o processamento da regra.

- `--property`

  Imprima também as propriedades do evento.

- `--subsystem-match=*`string[/string]`*`

  Filtrar eventos por subsistema[/devtype]. Somente eventos udev com um valor de subsistema correspondente serão permitidos.

- `--tag-match=*`string`*`

  Filtrar eventos por propriedade. Somente eventos udev com uma determinada tag anexada serão repassados.

- `--help`

  Imprimir texto de ajuda.

### teste udevadm [ opções ]*`devpath`*

Simule a execução de um evento udev para o dispositivo especificado e imprima a saída de depuração.

- `--action=*`string`*`

  A sequência de ações.

- `--subsystem=*`string`*`

  A cadeia do subsistema.

- `--help`

  Imprimir texto de ajuda.

### udevadm test-builtin [ opções ]*`command`* *`devpath`*

Execute um comando interno para o dispositivo especificado e imprima a saída de depuração.

- `--help`

  Imprimir texto de ajuda.

Autor

Escrito por Kay Sievers`<``kay.sievers@vrfy.org>` .

## Veja também

udev (7) udevd (8)

