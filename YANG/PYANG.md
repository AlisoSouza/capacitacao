# PYANG
# YANG
YANG é uma linguagem utilizada para descrever data models, Especialmente networking data models.  YANG é uma linguagem de fortemente tipada. Alguns aspectos da linguagem para levar em consideração: 
- Todo data model é um `module`, uma hierarquia _top level_ independente de nós (nodes).
- Data types podem ser `imported` de outros módulos YANG ou definidos dentro de um módulo.
- Usa `containers` para agrupar nodes relacionados
- Usa `lists` para identificar nodes que são guardados em sequência.
- Cada atributo individual de um node é representado por uma `leaf`.
- Cada leaf deve ter um `type` associado.
Exemplo mais aprofundado sobre modelagem de dados com YANG [aqui](https://github.com/AlisoSouza/capacitacao/blob/docs/YANG/yang-data-modeling.md).
# YANG data model
The YANG language is used to write **YANG data models**. An individual data model, called a "YANG Module" describes some aspect of networking.
A linguagem YANG é usada para escrever **YANG data models**.  um data model individual é chamado de "YANG module" e descreve algum aspecto de networking.
Data models podem descrever algum aspecto de um dispositivo de rede ou podem descrever um serviço de rede 
e utilizados por instrumentação de rede ou sistemas de gerenciamento para abstrair diferenças de dispositivos individuais para consumo do usuário final.

O model pode ser representado e apresentado em uma variedade de formatos, dependendo de qual é necessária no momento. Algumas opções são:
-   YANG language
-   Clear text
-   XML
-   JSON
-   HTML/JavaScript

Esses models podem ser escritos por qualquer um mas são usados com mais frequência os models de:
1. Comunidades/Grupos padronizadores como [IETF](https://www.ietf.org) e [Openconfig](https://www.openconfig.net).
2. Fornecedores de equipamentos e software de rede.

# Model Augmentation e Deviation
Os modelos YANG tem a habilidade de ser **augmented**(aumentados) ou **deviated from**(modificados) à medida que são criados ou quando implementados em uma solução. 
[Exemplo Cisco](https://developer.cisco.com/learning/modules/intro-device-level-interfaces/intro-yang/step/3)
[Exemplo na prática]()

# PYANG
Finalmente, o [pyang](https://github.com/mbj4668/pyang), é uma ferramenta bem útil capaz de validar, converter um YANG model. e gerador de código escrito em python.

[arquivo yang utilizado no exemplo](https://github.com/YangModels/yang/blob/master/vendor/cisco/xe/16111/ietf-interfaces.yang)
## Validação do YANG model
`pyang ietf-interfaces.yang`
Se o model não conter erros o terminal não apresentará um output. Caso contrário haverá um output informando erro.
Exemplo: 
Em `typedef interface-ref` apague o `path` e rode o comando novamente, um output informando o erro aparecerá:
nesse caso o output será: `ietf-interfaces.yang:55: error: a type leafref must have a path statement`
## Visualização em formato árvore
O pyang pode gerar uma representação do YANG model muito mais fácil de entender utilizando o comando:
	`pyang -f tree ietf-interfaces.yang`

Output esperado:
```yang
 module: ietf-interfaces  
   +--rw interfaces  
   |  +--rw interface [name]
   |     +--rw name                        string  
   |     +--rw description?                string  
   |     +--rw type                        identityref  
   |     +--rw enabled?                    boolean  
   |     +--rw link-up-down-trap-enable?   enumeration {if-mib}?  
   |     +--ro admin-status                enumeration {if-mib}?  
   |     +--ro oper-status                 enumeration  
   |     +--ro last-change?                yang:date-and-time  
   |     +--ro if-index                    int32 {if-mib}?  
   |     +--ro phys-address?               yang:phys-address  
   |     +--ro higher-layer-if_            interface-ref  
   |     +--ro lower-layer-if _interface-ref  
   |     +--ro speed?                      yang:gauge64  
   |     +--ro statistics  
   |        +--ro discontinuity-time    yang:date-and-time  
   |        +--ro in-octets?            yang:counter64  
   |        +--ro in-unicast-pkts?      yang:counter64  
   |        +--ro in-broadcast-pkts?    yang:counter64  
   |        +--ro in-multicast-pkts?    yang:counter64  
   |        +--ro in-discards?          yang:counter32  
   |        +--ro in-errors?            yang:counter32  
   |        +--ro in-unknown-protos?    yang:counter32  
   |        +--ro out-octets?           yang:counter64  
   |        +--ro out-unicast-pkts?     yang:counter64  
   |        +--ro out-broadcast-pkts?   yang:counter64  
   |        +--ro out-multicast-pkts?   yang:counter64  
   |        +--ro out-discards?         yang:counter32  
   |        +--ro out-errors?           yang:counter32  
   x--ro interfaces-state  
      x--ro interface_ [name]  
         x--ro name               string  
         x--ro type               identityref  
         x--ro admin-status       enumeration {if-mib}?  
         x--ro oper-status        enumeration  
         x--ro last-change?       yang:date-and-time  
         x--ro if-index           int32 {if-mib}?  
         x--ro phys-address?      yang:phys-address  
         x--ro higher-layer-if _interface-state-ref  
         x--ro lower-layer-if_    interface-state-ref  
         x--ro speed?             yang:gauge64  
         x--ro statistics  
            x--ro discontinuity-time    yang:date-and-time  
            x--ro in-octets?            yang:counter64  
            x--ro in-unicast-pkts?      yang:counter64  
            x--ro in-broadcast-pkts?    yang:counter64  
            x--ro in-multicast-pkts?    yang:counter64  
            x--ro in-discards?          yang:counter32  
            x--ro in-errors?            yang:counter32  
            x--ro in-unknown-protos?    yang:counter32  
            x--ro out-octets?           yang:counter64  
            x--ro out-unicast-pkts?     yang:counter64  
            x--ro out-broadcast-pkts?   yang:counter64  
            x--ro out-multicast-pkts?   yang:counter64  
            x--ro out-discards?         yang:counter32  
            x--ro out-errors?           yang:counter32
```
Sobre o output:
- O módulo ietf-interfaces provê dois containers:
	- interfaces
	- interfaces-state
- Dentro de cada container há uma lista chamada interface
	- A instância de uma interface é identificada por uma chave única do _[name]_
	- Cada atributo _leaf_ tem:
	- read-write (rw) ou read-only (ro)
	- Algum são opcionais (?)
	- data types definidos explicitamente
