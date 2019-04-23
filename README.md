# Como fazer loops no R?

---
title: "Como fazer loops no R?"
author:
- Ana Letycia B. Garcia^[PhD student <garcia.alb@usp.br>]
- Germano Martins F. Costa Neto^[PhD student <germano.cneto@usp.br>]
- Nathalia Salgado Silva^[PhD student <nathalia.salgado@usp.br>]
- Rafael Massahiro Yassue^[PhD student <rafael.yassue@usp.br>]
subtitle: Guia basico com exemplos
output:
  rmdformats::readthedown:
    highlight: kate
    self_contained: false
---

```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = FALSE)
data(iris)
require(plyr)
require(knitr)
require(rrBLUP)
require(snpReady)
```


## Estruturas de repetição 

### Estruturas de repetição ou *Loops*

Categoria de funções presente em toda e qualquer linguagem de 
programação cujo propósito consiste em **executar tarefas repetidas vezes**, até que algum critério seja satisfeito

### Conceitos-chave

Compreender as diferenças entre **até que** e **enquanto que**

### Principais diferenças

      for(contador in total) { funcao }
      
      while(condicao) { funcao }

      repeat{ funcao(condicao) }


## Estruturas de repetição: for(){}

####  Sintaxe Baseada em três elementos

- contador ou indexador (index)
- total de repetições a serem executadas (n)
- elementos, funções ou comandos a serem repetidos

### No ambiente R

      for (index in n)
       {
          funcoes
          comandos
       }
       
       ou 
       
       for(index in n){ funcoes }
       
## Estruturas de repetição: for(){}

### Exemplo prático

No dataset *iris*, deseja-se saber a média de cada uma das quatro características para cada uma das espécies (Setosa, Versicolor e Virginica).

```{r,echo=F}
kable(head(iris))
```

 - Como primeiro exemplo, faremos primeiro apenas para *Sepal.Length*

```{r,echo=TRUE}
n = nlevels(iris$Species)

for(i in 1:n){
  quais = which(iris$Species == levels(iris$Species)[i])
  m = mean(iris$Sepal.Length[quais])
  print(m)
}

```


- Agora para a combinação caracteristicas x espécies

```{r,echo=T}
# matriz de resultados
medias = matrix(NA, ncol = 4, nrow = 3)
colnames(medias) = colnames(iris)[1:4]
rownames(medias) = levels(iris$Species)

for(i in 1:nrow(medias)){
  for(j in 1:ncol(medias)){
    row = which(iris$Species == levels(iris$Species)[i])
    medias[i,j] = mean(iris[row,j])
  }
}

kable(medias)
```


## Busque também

### pacotes e funções (avançadas)

- plyr / dplyr packages
- apply(), tapply(), sapply, laply
- ddply, ldply, llply, adply etc

### Combinando **ddply** e **apply**

- ddply para *loop* de em *Species*
- apply para **aplicar** função média por coluna


- mesmo exemplo com dataset *iris*

```{r,echo=T}
medias=ddply(iris, .(Species), function(x) apply(x[,-5],2,mean))
kable(medias)
```

       
## Estruturas de repetição: while(){}

O *loop* do tipo *while* é executado até que dada condição seja satisfeita


### Baseada em dois elementos

- condição
- elementos, funções ou comandos a serem repetidos

### No ambiente R

      while(condicao) {
      funcao}
       
### Exemplos usando *while*

```{r,echo=T}
i=1
z=1

while(z < 30){
  z=(i+1)
  print(paste("z", z, sep = "="))
  i<-i+1
}
```
      
- outras aplicações: *Expectation Maximization* (EM)
- Aprendizado de parâmetros (iterativo)

## Aplicação de *loops* para solucionar problemas: **Predição Genômica**

### Capacidade preditiva de modelos

Deseja-se avaliar a capacidade preditiva de um modelo com base na correlação entre o valor fenotípico observado e predito $$\hat r = cor(y,\hat y)$$ 
Para isso provoca-se um desbalanceamento nos dados originais visando simular situações em que não há o fenótipo disponível na população de treinamento. Assim, buscamos predizer esse fenótipo com o modelo de interesse. Como sabemos $y$, podemos computar $\hat r$ por meio de sucessivos processos de reamostragem.

```{r,echo=F}
nomeoarquivo = "maize" #' usando arquivo maize

maize <- read.csv(paste("https://raw.githubusercontent.com/biomarkUSP/Datasets/master/",
                      nomeoarquivo,".csv",sep=""),header=T)



```

- Faça download do dataset *maize*  na nossa página no Git Hub (https://github.com/biomarkUSP/Datasets)

- 12 marcas em 171 individuos

```{r}
kable(head(maize))
```

### GBLUP (simpificado para fins didáticos)


$$ \mathbf{y} = \mathbf{\mu 1} + \mathbf{Z_g} + \mathbf{\varepsilon}$$

$\mathbf{g} \sim N (0,\mathbf{G})$ e $\varepsilon \sim N ( 0 , \Sigma \otimes I )$

- uso do pacote rrBLUP

Atributos inicias (expostos de forma didática)

```{r,echo=T}

r = NULL           # vetor nulo de correlacoes
y = maize$GY       # vetor de fenotipos
SNP = maize[,3:14] # matriz de SNPs
n = nrow(SNP)      # total de individuos
times = 100        # numero de vezes que vamos repetir o loop
sampled = n*.2     # 20% dos individuos

# ver pacote snpReady
G = G.matrix(SNP, method = "VanRaden", format = "wide", plot = F)$Ga

```


```{r,echo=T}

for(i in 1:times){
  
  #' fase de amostragem e producao do set de treinamento
  tst = sample(1:length(y), sampled, replace = F)
  ytst = y
  #' fase de producao do set de teste
  ytst[tst] = NA
  
  #' predicao genomica (GBLUP via rrBLUP)
   WGP = mixed.solve(y = ytst,K = G)
  
  #' correlcao entre y e yhat
   r[i] = cor(y[tst], WGP$u[tst])
}
```


```{r,echo=T}
mean(r)
sd(r)

boxplot(r)
```

