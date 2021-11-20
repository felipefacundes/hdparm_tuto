# Tradução em português do artigo da Linux Magazine de 2006 sobre HDPARM (atualizado).

## HDPARM
### Inspetor de disco: recuperando e definindo os parâmetros do disco rígido com hdparm

Hdparm é a ferramenta a ser usada quando se trata de ajustar seu disco rígido ou unidade de DVD, mas também pode medir a velocidade de leitura, fornecer informações valiosas sobre o dispositivo, alterar configurações importantes da unidade e até mesmo apagar SSDs com segurança.

Em 2005, o canadense Mark Lord desenvolveu o pequeno hdparm utilitário para testar drivers Linux para discos rígidos IDE. Desde então, o programa se tornou uma ferramenta valiosa para diagnóstico e ajuste de discos rígidos. Por exemplo, ele testa a velocidade dos discos rígidos e discos de estado sólido, coloca os dispositivos no modo de hibernação e ativa ou desativa o modo de economia de energia. Com dispositivos modernos, ele pode ativar o modo acústico e limpar SSDs. Antes de seus primeiros experimentos com hdparm, você deve ler sobre questões de segurança no “Aviso!” caixa.

### Necessidade de comunicação

Todas as distribuições razoavelmente novas já incluem hdparm na instalação básica. Você só precisa abrir um terminal e ligar

`hdparm -I /dev/sda`

como administrador (Figura 1).
[![Figura 1](https://www.linux-magazine.com/var/linux_magazin/storage/images/media/linux-magazine-eng-us/images/b01-hdparm1/544441-1-eng-US/b01-hdparm1_reference.jpg)]

```
Figura 1: Hdparm lista as propriedades de hardware de um disco rígido de seis anos com 320 GB de capacidade.
```

A ferramenta fornecerá todos os dados disponíveis sobre a unidade escolhida - neste caso, o primeiro disco rígido sda . O | mais opção garante que a grande quantidade de informações não passe simplesmente não lida pelo terminal.

O Hdparm aceita qualquer dispositivo como armazenamento em massa conectado a uma interface (E) IDE, SATA ou SAS, incluindo, portanto, unidades de DVD e SSDs. Os adaptadores USB para IDE geralmente causam problemas porque não transmitem os comandos ATA ou ATAPI (completos) para a unidade. As informações fornecidas pelo hdparm dependem do dispositivo. A designação e o número da versão do firmware estão sempre listados na parte superior, em Número do modelo e revisão do firmware . Os proprietários de um SSD, especialmente, podem descobrir rapidamente se estão executando a versão atual do firmware.

Em discos rígidos mais recentes, você deve verificar se o Native Command Queuing (NCQ) pode ser encontrado em Comandos / recursos . Essa tecnologia permite que o disco rígido classifique as consultas do sistema de forma que as cabeças percorram o caminho mais curto possível. Os SSDs, por outro lado, distribuem os acessos de gravação com mais eficiência entre os blocos de armazenamento. Idealmente, isso leva a um aumento na velocidade. Se o NCQ estiver desativado, verifique o BIOS para descobrir se a unidade está funcionando no AHCI modo , que também é necessário para outras funções, como gerenciamento de energia.

### Velocímetro

Para determinar a rapidez com que uma unidade entrega dados, use o

`hdparm -t /dev/sda`

comando. Após alguns segundos, a taxa de transferência de dados aparece (em megabytes por segundo, MBps). O pequeno programa lê diretamente da unidade por um tempo, independentemente do sistema de arquivos. A velocidade medida é, portanto, um pouco mais rápida do que na prática. Para receber um resultado não contaminado, nenhum outro programa deve estar em execução durante a medição, e memória principal suficiente deve estar livre. Repita a medição pelo menos três vezes e depois calcule o valor médio. Para um modelo atual, o resultado deve atingir pelo menos 80 MBps (Figura 2).

[![Figura 2](https://www.linux-magazine.com/var/linux_magazin/storage/images/media/linux-magazine-eng-us/images/b02-hdparm3/544444-1-eng-US/b02-hdparm3_reference.jpg)]
```
Figura 2: Este disco rígido SATA atingiu uma velocidade média de leitura de 80,48 MBps.
```

O kernel do Linux deposita os dados recuperados do disco rígido em um buffer. Para determinar a velocidade da unidade sem adornos, você pode usar o

`hdparm -t --direct /dev/sda`

comando. O Hdparm então lê os dados diretamente do disco. Os valores medidos assim serão um pouco mais lentos do que sem --direct , mas pelo menos você pode ver a taxa de transmissão pura do disco (Figura 3).

[![Figura 3](https://www.linux-magazine.com/var/linux_magazin/storage/images/media/linux-magazine-eng-us/images/b03-hdparm4/544447-1-eng-US/b03-hdparm4_reference.jpg)]
```
Figura 3: Sem o buffer, a taxa de transmissão cai drasticamente. No meio do disco rígido de 320 GB, mais perdas de velocidade são vistas.
```

O Hdparm sempre lê os dados desde o início do dispositivo de armazenamento. Os discos rígidos, no entanto, tendem a entregar dados um pouco mais lentamente das áreas externas dos discos magnéticos; portanto, o hdparm permite definir um deslocamento (a partir da versão 9.29 do software):

`hdparm -t --direct --offset 500 /dev/sda`

Os 500 representam o número de gigabytes a serem ignorados. Em um disco rígido de 1 TB, o comando acima, portanto, forneceria dados do meio do disco. Como mostra a Figura 3, a velocidade de leitura cai bastante nas áreas externas de um disco rígido.

Todos os testes de velocidade introduzidos aqui dão apenas uma primeira impressão de possíveis problemas e gargalos. Para um benchmark completo, entretanto, você também precisaria determinar a velocidade de gravação, por exemplo.

### Rápido rápido

Algumas propriedades da unidade podem ser alteradas enquanto o dispositivo está em operação; por exemplo, a maioria das unidades permite ativar e desativar o gerenciamento de energia. Apenas quais funções o hdparm pode alterar e ativar em um disco rígido podem ser acessadas com

`hdparm -I /dev/sda`

e são encontrados em Comandos / recursos (Figura 1). Todas as funções encontradas lá e marcadas com um asterisco estão atualmente ativas e o hdparm pode usar o resto ou pelo menos ativá-las.

Para acelerar a transmissão de dados, um disco rígido geralmente lê vários setores ao mesmo tempo. Quantos ele pode entregar ao mesmo tempo é revelado por

`hdparm -I /dev/sda`

e é listado após a transferência de setores múltiplos R / W: Máx = . Este valor também deve ser encontrado na mesma linha após Current = . Se não for o caso, você pode aumentar o valor com:

`hdparm -m16 /dev/sda`

Isso instrui o disco rígido a sempre fornecer 16 setores de uma vez.

Curiosamente, alguns discos rígidos funcionam mais devagar com valores mais altos: a página do manual hdparm menciona principalmente discos Caviar mais antigos da Western Digital. Nesses casos, você deve reduzir o número de setores novamente ou mesmo desligar completamente a função, o que é feito com:

`hdparm -m0 /dev/sda`

Além disso, as unidades modernas podem até mesmo recuperar alguns setores com antecedência (“leia mais adiante”). Para definir quantos, use a -a chave (Figura 4, parte superior) - por exemplo:

`hdparm -a256 /dev/sda`

[![Figura 4](https://www.linux-magazine.com/var/linux_magazin/storage/images/media/linux-magazine-eng-us/images/b04-hdparm6/544450-1-eng-US/b04-hdparm6_reference.jpg)]
```
Figura 4: aqui, a leitura antecipada é definida para 256 e o ​​gerenciamento acústico está desativado no momento.
```

Aqui, a unidade lerá os 256 setores com antecedência que são provavelmente os próximos a serem solicitados. Valores maiores aceleram acima de tudo a leitura de arquivos grandes - ao custo, entretanto, que a leitura de arquivos menores leva mais tempo. A configuração atual é mostrada com

`hdparm -a /dev/sda`

Além disso, muitas unidades também possuem uma função de leitura antecipada adicional embutida. Como regra, portanto, você pode deixar a configuração com o valor padrão.

A rapidez com que as consultas do sistema operacional chegam ao controlador do disco rígido pode ser acessada com

`hdparm -c /dev/sda`

O valor deve ser 32 bits ; você pode forçar esse valor com a -c3 opção .

### Velocidade máxima a frente

Muitos discos rígidos modernos permitem diminuir o movimento da cabeça. Embora isso aumente os tempos de acesso, também reduzirá o nível de ruído. Para descobrir se seu próprio disco rígido oferece esse “modo acústico”, você pode usar este comando:

`hdparm -M /dev/sda`

Se um número seguir o sinal de igual, conforme mostrado na Figura 4 (parte inferior), a unidade pode ser colocada em modo silencioso com:

`hdparm -M 128/dev/sda`

Para atingir a velocidade mais alta, use o valor máximo:

`hdparm -M 254 /dev/sda`

Valores entre 128 e 254 são permitidos, resultando em uma compensação entre o nível de ruído e a velocidade. A propósito, o kernel do Linux também deve suportar gerenciamento acústico, o que deve ser o caso para todas as principais distribuições atuais.

Algumas unidades de CD e DVD se parecem mais com turbinas: sua rotação em alta velocidade pode impedir a diversão de áudio / vídeo. o

`hdparm -E 4 /dev/sr0`

o comando proporcionará alívio. O parâmetro 4 determina a velocidade e / dev / sr0 especifica a unidade de DVD. Este exemplo diminui a velocidade de leitura da unidade em nove vezes.

### Cache de write-back

Com o cache de write-back, o disco rígido primeiro armazena os dados a serem gravados em um buffer. Dessa forma, ele pode aceitar dados muito mais rápido, o que no final leva a uma velocidade de gravação mais rápida. o

`hdparm -W /dev/sda`

o comando mostra se o cache de write-back está ativo com um 1 após o sinal de igual; caso contrário, você pode ativar a função com a -W1 chave .

Se o hdparm não permitir essa alteração, você precisa se certificar de que o cache de write-back foi ativado no BIOS. No entanto, esta função não é recomendada para todas as situações: No caso de uma queda de energia, os dados no buffer seriam perdidos permanentemente.

Se um programa sensível à perda de dados - como um banco de dados - estiver sendo executado no sistema, você deve desligar o cache de write-back com a -W0 opção . A documentação do banco de dados PostgreSQL até recomenda explicitamente que isso seja feito.

### Live Wire

Se um disco rígido ou SSD não tiver nada para fazer por um determinado período, ele entrará automaticamente no modo de hibernação. Este recurso de economia de energia pode ser influenciado com o -B parâmetro . Assim, usando:

`hdparm -B255 /dev/sda`

desativaria o gerenciamento de energia; entretanto, nem todas as unidades permitem isso.

Em vez de 255 , são permitidos valores entre 1 e 254. Um valor mais alto significa que mais potência é usada, mas também promete maior desempenho ou velocidade. Valores entre 1 e 128 permitem que o inversor desligue, enquanto valores de 129 a 254 proíbem que isso aconteça.

O máximo de energia pode ser economizado com um valor de 1 ; a taxa mais alta de transmissão de dados (desempenho de E / S) é alcançada com 254 . Você pode acessar o valor atual com:

`hdparm -B /dev/sda`

O efeito específico que os diferentes valores terão depende do próprio inversor. No entanto, você deve ter em mente que muitos desligamentos não são bons para discos rígidos de desktop: Cada vez que ele desliga, a unidade deve estacionar as cabeças, o que aumenta o desgaste. Conseqüentemente, você não deve ativar seu disco rígido a cada dois segundos - o que sempre leva mais de dois segundos para fazer.

Você pode definir quantos segundos de inatividade o disco rígido deve esperar antes de entrar em modo de espera com o

`hdparm -S 128 /dev/sda`

trocar; entretanto, este valor aqui não está em segundos, mas em um número entre 1 e 253. O disco rígido multiplica este valor por outro. O valor escolhido no exemplo, 128 , está entre 1 e 240, para o qual o inversor usa um fator de cinco. Conseqüentemente, ele desligaria após 640 segundos de inatividade.

De 241 para cima, o fator de multiplicação aumenta continuamente. Em 251, o período de espera aumentou para 5,5 horas. Em 253, o valor é predefinido pelo fabricante, geralmente entre oito e 12 horas. O valor 254 é omitido; em 255, a unidade aguardará 21 minutos e 15 segundos. Um valor de 0 desativará o modo de hibernação completamente. Para fazer o disco rígido hibernar imediatamente, digite:

`hdparm -y /dev/sda`

Com maiúsculo Y, a unidade entrará em um estado de hibernação ainda mais profundo. Dependendo da unidade, ela só pode acordar de um sono profundo após uma reinicialização de todo o sistema.

### Limpar

Os SSDs rastreiam a localização dos dados depositados neles, independentemente do sistema operacional. Isso pode levar à situação curiosa de que um arquivo foi excluído, mas o SSD ainda tem sua localização anterior marcada como ocupada. Para remediar esses conflitos, as versões mais recentes do hdparm incluem o wiper.sh script. Entrando

`wiper.sh /dev/sda`

determina quais blocos estão sendo usados ​​e quais não estão e relata isso ao SSD. No entanto, este script deve ser usado com cuidado: a documentação avisa explicitamente que os dados podem ser perdidos e não aconselha seu uso com o sistema de arquivos Btrfs. Unidades com ext2 / 3/4, Reiser3 e XFS devem ser montadas como somente leitura antes de usar o comando wiper. Seria melhor desmontar a unidade completamente ou iniciar o wiper.sh em um sistema Live. Em qualquer caso, você definitivamente deve fazer um backup do SSD com antecedência e usar o script apenas em uma emergência. A propósito, como o limpador é muito perigoso, algumas distribuições nem o incluem.

### Exclusão segura

Para atingir taxas de transferência mais altas e espalhar o uso igualmente pelos chips de armazenamento, os SSDs também reservam algumas áreas de armazenamento (nivelamento de desgaste), de forma que a simples formatação de um SSD raramente excluirá a unidade inteira. A maioria dos SSDs, portanto, oferece uma função chamada apagamento seguro , que faz com que a unidade esvazie todas as células de armazenamento. Isso é especialmente útil se você decidir desistir do SSD usado.

O apagamento seguro tem duas armadilhas: o hdparm só pode iniciar um apagamento seguro quando o BIOS também permite. Além disso, o método é considerado experimental. A documentação avisa explicitamente sobre o uso do procedimento porque, no pior dos casos, o apagamento seguro pode inutilizar todo o SSD. Se você quiser usar esta função de exclusão de qualquer maneira, primeiro chame as informações de identificação com:

`hdparm -I /dev/sdb`

Em Segurança , a linha suportada: o apagamento aprimorado deve aparecer em algum lugar; caso contrário, o SSD não oferecerá suporte ao apagamento seguro. Em seguida, ative a função de segurança da unidade configurando (temporariamente) uma senha como 123456 :

`hdparm --user-master u --security-set-pass 123456 /dev/sdb`

Ao chamar as informações de identificação novamente, você encontrará habilitado em Segurança . Para apagar o SSD agora, digite:

`hdparm --user-master u --security-erase 123456 /dev/sdb`

No processo, o hdparm também remove a senha. Todo o processo leva alguns minutos, dependendo do tamanho do SSD, durante o qual nenhum feedback é fornecido.

Posteriormente, quando você chamar as informações de identificação, a área em Segurança deve ficar novamente com a aparência anterior à configuração da senha.

### Relíquias

No caso de discos rígidos mais antigos com um conector IDE (também chamado de PATA), você deve dar uma olhada na using_dma linha na saída de identificação. Com a ajuda da tecnologia DMA (Direct Memory Access), o próprio disco rígido deposita os dados diretamente na memória principal. Se o respectivo sinalizador for 0 (desligado) , isso tornará a transferência de dados mais lenta. Com o passar dos anos, padrões de DMA cada vez mais rápidos foram introduzidos; o mais rápido possível pode ser ativado com o comando:

`hdparm -d1 /dev/hda`

Em alguns sistemas muito antigos, no entanto, o modo DMA pode causar problemas. Depois de ativá-lo, você deve copiar alguns arquivos de teste maiores para a unidade. Se surgirem problemas ou a unidade travar, desative o modo DMA novamente com:

`hdparm -d0 /dev/hda`

A propósito, as unidades SATA modernas sempre usam DMA.

Enquanto o disco rígido está transferindo os dados solicitados, o resto do sistema pode realizar outras tarefas - mas apenas se um on aparecer após unmaskirq na saída de informações de identificação. Você pode forçar este modo com a -u1 opção .

Valores Duradouros

Depois de reiniciar o sistema, todas as alterações feitas com hdparm são perdidas. Para ativá-los permanentemente, os respectivos comandos hdparm devem ser inseridos nos scripts de inicialização. Como isso é feito depende da distribuição que você está executando, mas normalmente a entrada deve ser feita em /etc/rc.local .

Os sistemas baseados em Debian, por outro lado, lêem o /etc/hdparm.conf arquivo de configuração na inicialização do sistema. Nele há uma seção para cada disco rígido com o seguinte formato:

```
/dev/sda {
   ...
 }}
```

Os sistemas Linux modernos alocam nomes de dispositivos aleatoriamente ( sda , sdb ). Para atribuir as configurações do hdparm a uma unidade específica permanentemente, use seu UUID específico:

`/dev/disk/by-id/ata-SAMSUNG_HD103SJ_S246J1RZB00034 {...}`

As configurações pertencem entre as chaves. Cada parâmetro tem seu próprio nome. O gerenciamento acústico é definido, por exemplo, para o valor de 128 com:

gerenciamento_acústico = 128

Qual nome pertence a qual parâmetro hdparm é revelado pelos comentários na parte superior do arquivo.

### Conclusões

O Hdparm também inclui muitos outros parâmetros que podem ser bastante perigosos. Por exemplo, muitos SSDs podem ser protegidos com uma senha, o que pode levar à perda de dados em algumas situações. Não é por acaso que a página do manual ( man hdparm ) avisa sobre os perigos.

Aliás, o hdparm é apenas uma ferramenta útil entre muitas; por exemplo, o smartmontools pode determinar o estado de saúde de um disco rígido.


Fonte: [Linux Magazine](https://www.linux-magazine.com/Online/Features/Tune-Your-Hard-Disk-with-hdparm)
