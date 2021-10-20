
---
output:
  pdf_document: default
  html_document: default
---


# Ambientes e caminho de busca

Na seção sobre funções, discutimos dois pontos interessantes sobre objetos, que é a atribuição e o escopo. E esses dois pontos estão intimamente relacionados ao objeto ambiente, de modo "environment". O ambiente é um objeto que armazena, em forma de lista, as ligações dos nomes associados aos objetos. Porém, existem diferenças entre o objeto lista do objeto ambiente, com quatro exceções (Wickham (2019)):

- Cada nome deve ser único;
- Os nomes em um ambiente não são ordenados;
- Um ambiente tem um pai ou também chamado de ambiente superior;
- Ambientes não são copiados quando modificados.

Muitas dessas definições são complexas para esse momento. E uma profundidade sobre o assunto, será abordada no módulo Programação em R (Nível Intermediário). Contudo, introduziremos algumas características importante os objetos ambiente.

O ambiente de trabalho do R é conhecido como ambiente global, pois é onde todo o processo de interação da linguagem ocorre. Existe um nome específico associado a esse objeto, que é .GlobalEnv, ou também pode ser acessado pela função globalenv(). Para sabermos quais os nomes que existem nesse ambiente, usamos a função ls(), isto é


```r
# Nomes no ambiente global
ls()
```

```
## character(0)
```

Quando o resultado da função é character(0), significa que não existem nomes criados no ambiente global. O ambiente corrente de trabalho é informado pela função environment(), isto é,

```r
# Comparando os ambientes
identical(environment(), .GlobalEnv)
```

```
## [1] TRUE
```

```r
# Forma errada de comparar ambientes (Erro...)
environment() == .GlobalEnv
```
Error in environment() == .GlobalEnv: comparação (1) é possível apenas para tipos lista ou atômicos

A segunda forma é equivocada, porque os ambientes não são vetores. Assim, como também não podemos utilizar o sistema de indexação, isto é,


```r
# Criando objetos no ambiente global
b <- 2; a <- "Ben"; x <- TRUE
# Verificando os nomes no ambiente global
ls()
```

```
## [1] "a" "b" "x"
```


```r
# Acessando o objeto "a"
.GlobalEnv$a
```

```
## [1] "Ben"
```


```r
.GlobalEnv[["a"]]
```

```
## [1] "Ben"
```

```r
# Acessando o primeiro nome (Erro...)
.GlobalEnv[[1]]
```

Error in .GlobalEnv[[1]]: argumentos errados para obtenção de subconjuntos de um ambiente

A última linha de comando retorna um erro, porque os nomes em ambientes não são ordenados, ao invés, devemos chamar os resultados por meio de $ ou .

Poderemos criar um ambiente pela função new.env(), e inserir ligações dentre dele, como apresentado a seguir.


```r
# Criando objetos no ambiente global
b <- 2; a <- "Ben"; x <- TRUE
# Verificando os nomes no ambiente global
ls()
```

```
## [1] "a" "b" "x"
```

```r
# Criando um objeto ambiente no ambiente global
amb1 <- new.env()
# Inserindo nomes nesse no ambiente "amb1"
amb1$d <- 3; amb1$e <- "FALSE"
# Verificando nomes no ambiente global
ls()
```

```
## [1] "a"    "amb1" "b"    "x"
```


```r
# Verificando nomes no ambiente "amb1"
ls(envir = amb1)
```

```
## [1] "d" "e"
```

Todo ambiente tem um ambiente pai ou ambiente superior. Quando um nome não é encontrado no ambiente corrente, o R procurará no ambiente pai. Para saber, use parent.env(), isto é,


```r
parent.env(amb1)
```

```
## <environment: R_GlobalEnv>
```

O único ambiente que não tem pai é o ambiente vazio, objeto emptyenv(), que pode ser observado pela linha de comando:

```
parent.env(emptyenv())
```
Error in parent.env(emptyenv()): o ambiente vazio não tem pai

## A superatribuição

A atribuição (<-) é uma função que associa um nome a um objeto no ambiente corrente. Quando usamos o R, quase sempre esse ambiente é o ambiente global. A superatribuição (<<-) cria um nome e o associa a um objeto no ambiente pai do ambiente de onde essa associação está sendo criada. Vejamos,


```r
# Criando o objeto x e o imprimindo
x <- 0; x
```

```
## [1] 0
```



```r
# Criando uma funcao com a superatribuicao
f1 <-  function() {
  # Obj2
  x <- 1
  # Modificando x do ambiente global
  x <<- 2
  # Imprimindo o ambiente de execucao
  env <- environment()
  # Imprimindo o Obj2
  res <- list(x = x, "Ambiente de execução" = env, "Ambiente Pai" = parent.env(env))
  # Retornando a lista
  return(res)
}
# Imprimindo f1
f1()
```

```
## $x
## [1] 1
## 
## $`Ambiente de execução`
## <environment: 0x000000001601fd68>
## 
## $`Ambiente Pai`
## <environment: R_GlobalEnv>
```


```r
# Imprimindo x
x
```

```
## [1] 2
```



```r
# Imprimindo o ambiente envolvente de f1
environment(f1)
```

```
## <environment: R_GlobalEnv>
```



```r
# Imprimindo os nomes do ambiente global
ls()
```

```
## [1] "a"    "amb1" "b"    "f1"   "x"
```

Esse caso é interessante porque vemos dois nomes associados a objetos em ambientes diferentes. Alguns ambientes são criados pela função function(), são os chamados ambientes funcionais. Um deles é o ambiente envolvente, já comentado na seção sobre funções. O ambiente envolvente da função f1 é o ambiente global. Já no corpo da função f1, um outro ambiente surge quando a função é chamada, é o ambiente de execução. Toda vez que a função é chamada, cria-se um novo ambiente de execução. Observemos os identificadores, em Ambiente de execução, quando executamos a função mais de uma vez,



```r
f1()$`Ambiente de execução`
```

```
## <environment: 0x0000000013ad0f38>
```


```r
f1()$`Ambiente de execução`
```

```
## <environment: 0x0000000015ec5078>
```


```r
f1()$`Ambiente de execução`
```

```
## <environment: 0x000000001634ff80>
```

O ambiente pai do ambiente de execução, é o ambiente envolvente de f1, que nesse caso é o ambiente global. Assim, observe que o ocorre quando executamos o comando de superatribuição. O nome x no ambiente global passou a está associado ao valor 2, porque foi alterado por <<-, mas o nome x continuou associado ao valor 1, porque a função f1() retornou o valor 1. Isso mostra que a superatribuição não cria um objeto no ambiente atual, mas em um ambiente pai se não existe ou altera o nome existente. Vejamos o complemento dessa afirmação no próximo exemplo.


```r
# Verificando os nomes no ambiente global
ls()
```

```
## [1] "a"    "amb1" "b"    "f1"   "x"
```


```r
# Criando uma funcao
f2 <-  function() {
  x <<- 2
}
# Executando f2
f2()
# Verificando novamente os nomes no ambiente global
ls()
```

```
## [1] "a"    "amb1" "b"    "f1"   "f2"   "x"
```


```r
# Verificando o valor de x
x
```

```
## [1] 2
```

Com a superatribuição executada dentro de f2() e como no ambiente pai não eistia o nome x, este foi criado e associado ao valor 2. Um próximo exemplo, consideramos um ambiente envolvente que não seja o ambiente global. Vejamos,


```r
# Funcao contador
contador <- function() {
  i <- 0
  env1 <- environment()
  aux <- function() {
    # do something useful, then ...
    i <<- i + 1
    env2 <- environment()
    res2 <- list(i = i, `AmbExec_aux` = env2, `AmbExec_contador` = env1)
    return(res2)
  }
}
# Chamada de funcao
contador1 <- contador()
contador1()
```

```
## $i
## [1] 1
## 
## $AmbExec_aux
## <environment: 0x0000000016395228>
## 
## $AmbExec_contador
## <environment: 0x000000001633aa38>
```


```r
contador1()
```

```
## $i
## [1] 2
## 
## $AmbExec_aux
## <environment: 0x00000000144ebeb8>
## 
## $AmbExec_contador
## <environment: 0x000000001633aa38>
```


```r
contador1()
```

```
## $i
## [1] 3
## 
## $AmbExec_aux
## <environment: 0x00000000141ee9d0>
## 
## $AmbExec_contador
## <environment: 0x000000001633aa38>
```



```r
# Chamada de funcao
contador2 <- contador()
contador2()
```

```
## $i
## [1] 1
## 
## $AmbExec_aux
## <environment: 0x0000000015ea8e88>
## 
## $AmbExec_contador
## <environment: 0x0000000014614270>
```

Quando uma função function() é criada dentro de outra função function() o ambiente de execução da função superior, contador(), é o ambiente envolvente da função interna, aux(). Dessa forma, o ambiente de execução de contador() não será mais efêmero, isto é, não será apagado após a execução, como pode ser visto em contador1(). Observamos que executamos contador1() três vezes. O nome i foi atualizado, devido a superatribuição, a cada chamada da mesma função. Ao passo que, quando realizamos uma nova chamada de contador(), por meio de contador2(), o resultado de i retora o valor 1, porque um novo ambiente de execução para contador() foi criado, como pode ser observado.

Os demais ambiente funcionais e exemplos, serão descritos no módulo Programação em R (Nível Intermediário).

Por fim, uma última forma do R encontrar os nomes é pelo **caminho de busca**, que além dos ambientes criados e o ambiente global, existem os ambientes de pacotes. Toda vez que um pacote é anexado ao caminho de busca, o ambiente de pacote anexado será sempre o pai do ambiente global. Vejamos,


```r
# Caminho de busca
search()
```

```
## [1] ".GlobalEnv"        "package:stats"     "package:graphics" 
## [4] "package:grDevices" "package:utils"     "package:datasets" 
## [7] "package:methods"   "Autoloads"         "package:base"
```




```r
# Anexando o pacote SMR
library(SMR)
# Verificando o caminho de busca
search()
```

```
##  [1] ".GlobalEnv"        "package:SMR"       "package:stats"    
##  [4] "package:graphics"  "package:grDevices" "package:utils"    
##  [7] "package:datasets"  "package:methods"   "Autoloads"        
## [10] "package:base"
```




```r
# Carregando o pacote midrangeMCP
library(midrangeMCP)
# Verificando o caminho de busca
search()
```

```
##  [1] ".GlobalEnv"          "package:midrangeMCP" "package:SMR"        
##  [4] "package:stats"       "package:graphics"    "package:grDevices"  
##  [7] "package:utils"       "package:datasets"    "package:methods"    
## [10] "Autoloads"           "package:base"
```

A lista dos ambientes no caminho de busca segue a ordem hierárquica dos ambientes, de modo que o ambiente global será sempre o ambiente de trabalho, isto é, o ambiente corrente. Não foi apresentado nessa lista, o ambiente vazio, emptyenv(). Mas, poderemos utilizar o pacote rlang para isso.



```r
# Criando um ambiente
amb2 <- new.env()
# Verificando seus parentais
rlang::env_parents(env = amb2, last = emptyenv())
```

```
##  [[1]] $ <env: global>
##  [[2]] $ <env: package:midrangeMCP>
##  [[3]] $ <env: package:SMR>
##  [[4]] $ <env: package:stats>
##  [[5]] $ <env: package:graphics>
##  [[6]] $ <env: package:grDevices>
##  [[7]] $ <env: package:utils>
##  [[8]] $ <env: package:datasets>
##  [[9]] $ <env: package:methods>
## [[10]] $ <env: Autoloads>
## [[11]] $ <env: package:base>
## [[12]] $ <env: empty>
```

Dessa forma, é caminho de busca que o R procurará pelos nomes. Isso significa, que se o ambiente envolvente de uma função, por exemplo, for o ambiente vazio, o R procurará pelas funções básicas no pacote base e não será encontrado, pois nesse ambiente não há nomes e nem ambientes parentais. Por isso, que o R depende do escopo léxico para tudo. Vejamos,


```r
# Criando uma funcao
f3 <- function() x + 1

# Modificando o ambiente envolvente de f3
environment(f3) <- emptyenv()

# Dependencias externas da funcao f3
codetools::findGlobals(f3)
```

```
## [1] "+" "x"
```



```r
# Chamando a funcao f3
f3()
```
Error in x + 1: não foi possível encontrar a função "+"

Isso não ocorre quando definimos o ambiente envolvente de f3() como sendo o ambiente global, porque quando a função buscar pelo operador de soma neste ambiente e não encontrar, f3() seguirá até o ambiente package:base, para encontrar o operador “+.”

Por isso, que usar a função attach() para anexar objetos do tipo quadro de dados (data frames), por exemplo, pode se tornar um problema em um código, quando temos nomes iguais para objetos diferentes. Para isso, apresentamos o código a seguir.


```r
# objeto quadro de dados
dados <- data.frame(sd = 1:3, var = (1:3)^2)
# Caminho de busca
search()
```

```
##  [1] ".GlobalEnv"          "package:midrangeMCP" "package:SMR"        
##  [4] "package:stats"       "package:graphics"    "package:grDevices"  
##  [7] "package:utils"       "package:datasets"    "package:methods"    
## [10] "Autoloads"           "package:base"
```



```r
# anexando "dados" ao caminho de busca
attach(dados)
# Verificando novamente o caminho de busca
search()
```

```
##  [1] ".GlobalEnv"          "dados"               "package:midrangeMCP"
##  [4] "package:SMR"         "package:stats"       "package:graphics"   
##  [7] "package:grDevices"   "package:utils"       "package:datasets"   
## [10] "package:methods"     "Autoloads"           "package:base"
```




```r
# Imprimindo sd
sd
```

```
## [1] 1 2 3
```




```r
# Desanexando "dados"
detach(dados)
# Imprimindo sd
sd
```

```
## function (x, na.rm = FALSE) 
## sqrt(var(if (is.vector(x) || is.factor(x)) x else as.double(x), 
##     na.rm = na.rm))
## <bytecode: 0x0000000015bcea70>
## <environment: namespace:stats>
```


Quando criamos o objeto dados, uma de suas colunas estava nomeada por sd, que também é o nome de uma função do pacote stats, que representa a variância. Porém, quando anexamos o objeto dados no caminho de busca, um novo ambiente é criado, como pai do ambiente global e com mesmo nome do objeto, e os elementos do objeto dados são copiados para esse ambiente. Assim, o nome sd foi procurado e não encontrado, seguindo a busca para o próximo ambiente que foi dados, e daí foi encontrado. Percebemos que também existe esse nome no ambiente do pacote stats, que é do tipo função. Entretanto, como o primeiro objeto encontrado associado a esse nome estava no ambiente dados, ele é retornado. Nesses casos, se usarmos a superatribuição, a alteração ocorrerá apenas na cópia dos elementos no ambiente anexado, e não nos elementos do objeto original. Caso haja a atribuição, o nome será criado no ambiente global.

Isso pode acabar se tornando um problema se muitos objetos forem anexados. Por isso, é preferível o uso da indexação ou $ para acessar os elementos de uma lista ou quadro de dados, evitando assim, conflitos na procura de nomes.

Desse mesmo modo, poderíamos pensar que esses mesmo conflitos poderiam surgir dentro de um pacote. Porém, graças ao NAMESPACE, isso não ocorre, sendo um dos assuntos abordados nos próximos módulos.