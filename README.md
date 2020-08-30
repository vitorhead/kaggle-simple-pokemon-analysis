---
title: "Programando IA com R"
subtitle: 14IA - Vitor Stipanich de Santiago | RM 339165
output:
  html_document:
    df_print: paged
  pdf_document: default
font-family: Roboto
---
###### Esse notebook foi desenvolvido visando o conteúdo coberto pelas aulas de R do professor Elthon, ministradas no curso de I.A.
<hr />

```{r, echo=FALSE,warning=FALSE,message=FALSE,error=FALSE}
library(tidyverse)
library(DT)
library(fmsb)
``` 


# O que é o R?

O R é uma linguagem muito potente por ser dinâmica e _data-driven_ . 
É uma linguagem baseada fortemente em **vetores, matrizes e listas**, já que são tipos de dados utilizados fortemente em análises estatísticas.

No console R, podemos digitar expressões e elas são avaliadas em real time:

```{r, warning=FALSE,message=FALSE,error=FALSE}
numeros = 1:10
print(numeros)
```

Neste trecho acima estou criando uma lista que vai de 1 até 10, de forma dinâmica através da função (que parece até um operador pela notação!) ** : ** .
Temos diversas maneiras de operar em cima dessas estruturas. Não precisamos realizar loops com condicionais embutidas para filtrar elementos de um vetor:

```{r, warning=FALSE,message=FALSE,error=FALSE}
pares = numeros[numeros %% 2 == 0]
print(pares)
```

E isso não se aplica apenas ao _mod_: podemos executar diferentes operadores ou funções que produzam resultados booleanos para filtrar os elementos dos vetores. Isso nos permite realizar diversas operações em grandes datasets, facilitando o dia a dia do cientista de dados.


Podemos criar matrizes? Podemos!
```{r}
minha.matriz <- matrix(data=seq(1, 1000), nrow=100, ncol=10)
head(minha.matriz, 5)
```
Observe que o ponto (".") no nome da variável não indica o atributo ou propriedade de um objeto, mas é um caractere comum! O nome da matriz que eu criei então é "minha.matriz", e não um objeto chamado "minha" com uma propriedade "matriz" dentro.

O R possui, nativamente, algumas funções para plotar gráficos e desenhar relações. Na prática, existem bibliotecas que fazem isso de maneira mais fácil (ou até mesmo iterativa) pra você. 
Vamos ver um simples scatterplot em R:
```{r}
plot(minha.matriz)
```

Apenas chamando a função _plot_ com a matriz criada o R já tenta assumir diversas coisas pra facilitar pra você.
Veja que a relação plotada foi de minha.matriz[,1] e minha.matriz[,2], já que não informamos nenhum valor específico para X e Y, cor dos pontos, linhas, etc. Vamos fazer isso?

```{r}
plot(x = minha.matriz[,1],
     y = minha.matriz[,2]*-1,
     type = "b",
     xlab = "Primeira coluna da minha matriz",
     ylab = "Décima coluna da minha matriz (valores negativos)",
     col="darkblue",
     lwd=10)
```

Nessa versão de plot, especifiquei os valores que quero pra X e Y (note que até modifiquei), os nomes, a largura  e cor dos pontos e mais. 
O R nativo é recheado de funções para trabalhar o dataset e plotar gráficos, sendo uma excelente ferramenta para análise de dados. Vamos ver abaixo algumas operações utilizando tanto R nativo quanto pacotes da comunidade.

Mas não vamos ficar apenas em vetores numéricos e listas mockadas...

# Operações em datasets

Ao invés de criar vetores numéricos e datasets simples para mostrar um pouco a linguagem, vamos partir para algo mais divertido.
Irei utilizar [esse dataset de Pokemon](https://www.kaggle.com/rounakbanik/pokemon) para mostrar um pouco do que o R pode fazer, junto de pacotes que são amplamente utilizados na indústria.

Primeiro, vamos carregá-lo:
```{r, warning=FALSE,message=FALSE,error=FALSE}
pokemon = read.csv("C:/Users/Vitor Stipanich/Documents/AI&ML/R/programando IA com R/pokemon.csv", stringsAsFactors=TRUE)
```

O comando acima, como é possível imaginar, lê o arquivo indicado no path como CSV, e você ainda pode passar diversos parametros para customizar a leitura.
Mas você mesmo já deve ter imaginado que seria problemático deixar o path fixo sem antes poder disponibilizar uma opção de baixar ou salvar o arquivo em um diretório diferente.<br/>
Vamos então pegar da web! <br/>
Para isso, vou definir uma função que faz o download de um arquivo por URL:
```{r, warning=FALSE,message=FALSE,error=FALSE}
baixar.arquivo.data <- function(file.url, file.local = NA, downloaded.file.name=NA) {
  if (is.na(file.local)) {
    file.local = file.path('./data', basename(file.url))
  } else {
    if (is.na(downloaded.file.name)) {
      file.local = file.path(file.local, basename(file.url))
    } else {
      file.local = file.path(file.local, downloaded.file.name)
    }
  }
  download.file(url = file.url, destfile = file.local, mode='wb', method='libcurl')
}

baixar.arquivo.data(file.url="https://gist.githubusercontent.com/vitorhead/133233cd37cc6af8d4e63582bf82e690/raw/82304c9dae9d5e944680d1bca97d883dbf386a7c/pokemon.csv",
                    file.local=paste(getwd(), "/data", sep=""),
                    downloaded.file.name="pokemon.csv")

# Para utilizarmos futuramente na leitura
downloaded.file.path = paste(getwd(), "data", "pokemon.csv", sep="/")
```

Nesse trecho acima criamos uma função com tres parâmetros, o segundo opcional para o programador salvar em um path específico. O terceiro parametro também é opcional, possibilitando que o arquivo baixado assuma outro nome para ser salvo. Agora podemos trabalhar com arquivos pela web e não ter desculpas de "na minha máquina funciona!!"<br/>
Agora sim, vamos ler o arquivo!

```{r, warning=FALSE,message=FALSE,error=FALSE}
pokemon = read.csv(downloaded.file.path, stringsAsFactors=TRUE, encoding="UTF-8")

pokemon %>% 
  head(20) %>% 
  select(name, type1, type2, hp, attack, defense) 
```

<hr />
O dataset foi parseado na primeira linha, e após isso utilizamos o comando pipe (%>%) para encadear as operações, gerando um código mais idiomático.
O head pega as primeiras 20 linhas, o select seleciona apenas as colunas informadas e o View mostra o dataset.

# Explorando

Temos um dataset com os 801 pokemon e diversas informações sobre os status (ataque, vida, defesa, velocidade, etc), tipos, habilidades e outros.
Vamos ver o que podemos extrair desses dados!

<hr />
#### Quantos pokemon existem por geração?
```{r, warning=FALSE,message=FALSE,error=FALSE}
pokemon %>% 
  group_by(generation) %>%
  summarize(quantidade=n()) 
```

<hr />
#### Podemos separar os pokemon por tipo primário (type1)?
```{r, warning=FALSE,message=FALSE,error=FALSE}
pokemon %>% 
  group_by(type1) %>%
  summarize(quantidade=n())
```

#### Tabelas não! Vamos fazer um gráfico pra visualizar melhor a divisão por tipos:

```{r, fig.width=10, fig.height=8, warning=FALSE,message=FALSE,error=FALSE}
# Primeiro vamos definir uma matriz que guarda o nome do tipo e uma cor para representa-lo
# Assim nosso grafico fica bonito :)
type_colors <-
  matrix(data = c(
  	c("Bug", "#A8B820"),
  	c("Dark", "#705848"),
  	c("Dragon", "#7038F8"),
  	c("Electric", "#F8D030"),
  	c("Fairy", "#EE99AC"),
  	c("Fighting", "#C03028"),
  	c("Fire", "#F08030"),
  	c("Flying", "#A890F0"),
  	c("Ghost", "#705898"),
  	c("Grass", "#78C850"),
  	c("Ground", "#E0C068"),
  	c("Ice", "#98D8D8"),
  	c("Normal", "#A8A878"),
  	c("Poison", "#A040A0"),
  	c("Psychic", "#F85888"),
  	c("Rock", "#B8A038"),
  	c("Steel", "#B8B8D0"),
  	c("Water", "#6890F0")), nrow = 18, ncol = 2, byrow=TRUE)


qtd.type <- pokemon %>% 
  group_by(type1) %>%
  summarize(quantidade=n()) 


# angulos pra posicionar
labels.type.plot <- qtd.type
number_of_bar <- nrow(labels.type.plot)
angle <-  90 - 360 * (seq(1,length(labels.type.plot$type1))-0.5) / number_of_bar     

labels.type.plot$hjust<-ifelse( angle < -90, 1, 0)

# girar o angulo pra deixar o texto legivel
labels.type.plot$angle<-ifelse(angle < -90, angle+180, angle)

ggplot(qtd.type, aes(x=as.factor(type1), y=quantidade)) +
  geom_bar(stat="identity", fill=type_colors[1:18,2]) +
  ylim(-80,170) + # valor negativo = circulo interno, positivo = tamanho de cada barra
  geom_text(data=labels.type.plot, aes(x=type1, y=quantidade+10, label=paste(type1, quantidade, sep=" - "), hjust=hjust), color="black", fontface="bold",alpha=0.6, size=4, angle=labels.type.plot$angle) +
  theme_minimal() +
  theme(
    axis.text = element_blank(),
    axis.title = element_blank(),
    panel.grid = element_blank(),
    plot.margin = unit(rep(-1,4), "cm")
  ) +
  coord_polar(start = 0)
```

<hr />
#### Qual o pokemon do tipo grama que tem o maior ataque?
```{r, warning=FALSE,message=FALSE,error=FALSE}
pokemon %>% 
  filter(type1 == "grass") %>% 
  arrange(desc(attack)) %>% 
  select(name, generation, attack, hp, defense, type1, type2, is_legendary) %>% 
  head(1) 
```

O pokemon com maior ataque é Kartana. Mas é um pokemon lendário, assim como visto na última coluna.

<hr />
#### Podemos descobrir o pokemon mais rápido, porém dessa vez fazendo distinção dos lendários e não lendários?
```{r, warning=FALSE,message=FALSE,error=FALSE}
pokemon %>% 
  group_by(is_legendary) %>%
  top_n(1, speed) %>% 
  select(name, generation, attack, hp, defense, speed, type1, type2, is_legendary) 
```

<hr />
#### Dividindo os pokemon por tipagem primária (type1), qual é o mais pesado (kg) de cada tipo?
```{r, fig.width=9, fig.height=10, warning=FALSE,message=FALSE,error=FALSE}
pokemon %>% 
  group_by(type1) %>% 
  top_n(1, weight_kg) %>% 
  select(name, type1, weight_kg) %>% 
      ggplot(aes(x=type1, y=weight_kg)) +
      geom_col(aes(fill=type1), position = "dodge", width=0.5) +
      theme(axis.text.x=element_text(angle = 35, hjust = 1)) +
      labs(x="Tipo", y="Peso (kg)", fill="Tipos") +
      scale_fill_manual(values=type_colors[,2]) +
      geom_text(
        aes(label = name),
        position = position_stack(vjust = 0.5),
        angle=90,
        colour="white")
```


## Batalhas

Os jogos pokemon são muito famosos por possibilitar a chance de construir um time com os seus favoritos e batalhar vários treinadores durante a sua jornada até a liga. Durante uma batalha, os pokemon utilizam ataques físicos ou especiais para infligir dano no seu oponente. Cada ataque tem um tipo, e os tipos possuem vantagens e desvantagens entre si. <br/>
Nesse dataset, as colunas **against_XXXX** (onde XXXX é o tipo) representam o quão efetivo aquele tipo é sobre o pokemon correspondente. <br/>
**Esse dataset é bem simples; não contém informações sobre ataques nem informações sobre se o pokemon é o último estágio evolucionário de sua linha. Iremos fazer operações simples apenas visando o ferramental do R!**
<hr />
Será que conseguimos descobrir, por exemplo, qual(is) seria(m) uma boa escolha para enfrentar o pokemon Hypno?
Vamos começar recuperando apenas ele do dataset:
```{r}
hypno = pokemon %>% filter(name == "Hypno")
```
Agora, para descobrir as fraquezas, vamos recuperar todas que sejam maior que 1.0 (dano neutro):
```{r}
fraquezas.hypno = hypno %>% 
                  select(contains("against_")) %>%
                  select_if(~any( . > 1.0))
fraquezas.hypno
```

Vamos então recuperar uma lista de pokemon que contenham um ou mais dessas tipagens como type1 ou type2:
```{r}
# Removendo o agaisnt para utilizar como filtro
fraquezas.hypno.formatado = 
  map(colnames(fraquezas.hypno), function(colname) { str_replace(colname, "against_", "") })

oponentes.hypno = pokemon %>% 
                      filter(is.element(type1, fraquezas.hypno.formatado) | is.element(type2, fraquezas.hypno.formatado) ) %>% 
                      select(name, type1, type2, generation, attack, defense, hp) %>% 
                      arrange(desc(attack)) 
```

Esses então seriam os pokemon que, teoricamente, teriam uma vantagem contra o Hypno em uma batalha.<br/>
_"Teoricamente"_ pois em uma batalha no jogo diversos eventos podem acontecer, como o uso de itens ou a mudança de pokemon em seu turno!
<br/>
<hr/>

```{r}
#utils
vetor.atributos <- function() {
  return(c("attack", "defense", "hp", "speed", "sp_attack", "sp_defense"))
}

print.datatable.treinador <- function(treinador, nome) {
  datatable(treinador, 
            caption = htmltools::tags$caption(
              style = 'caption-side: top; text-align: center; font-size: 20pt;',
              nome
            ),
            class = 'cell-border stripe', 
            rownames = FALSE, 
            options = list(dom = 't'), 
            filter = list(position = "none"))    
}
```

Neste seção acima estou definindo duas funções utilitárias para usarmos daqui pra frente nas próximas análises.
O intuito da primeira é agregar algumas colunas em um vetor para ficar mais fácil de chamar no select() do pacote dplyr.<br/>
Já a segunda função nos auxilia a printar um datatable do pacote DT com uma estética mais definida.


## Criando times

Outra situação bem comum no jogo é encontrar treinadores especializados em tipos ou temáticas de pokemon. <br/>
Com um pouco de criatividade e esse dataset, podemos criar situações parecidas:
```{r}
treinador.joey <- pokemon %>% 
                    filter(type1 == "water" & 
                           (attack >= 100 | sp_attack >= 100) & 
                           speed >= 75 & 
                           is_legendary == 0) %>% 
                    select(name, type1, type2, generation, vetor.atributos(), against_bug:against_water) %>% 
                    sample_n(6)
print.datatable.treinador(treinador.joey[,1:10], 'Treinador Joey')                                      
```

No código acima, criei a variável representando o treinador Joey, que gosta de pokemon do tipo água.<br/>
Os pokemon de Joey são fortes e rápidos (por isso o filtro no ataque, ataque especial e velocidade) e ele também não possui pokemon lendários.
<br />
<br />
<br />

Vamos agora criar outro treinador, dessa vez não se restringindo os tipos:
```{r}
treinador.steve <- pokemon %>% 
                      filter(hp >= 100 & defense >= 110 & is_legendary == 0) %>% 
                      select(name, type1, type2, generation, vetor.atributos(), against_bug:against_water) %>% 
                      sample_n(6)
print.datatable.treinador(treinador.steve[,1:10], 'Treinador Steve')                 
```



Será que é possível criarmos uma função que receba o nome de um pokemon e descubra um bom parceiro para uma batalha em dupla, levando em consideração suas fraquezas e seus atributos base?

```{r}
check.pokemon <- function(nome) {
  return(any(pokemon$name == nome))
}

find.pokemon.weakness <- function(nome) {
   pokemon.escolhido <- pokemon[pokemon$name == nome,]
   fraquezas <- pokemon.escolhido %>% 
                  select(contains("against_")) %>%
                  select_if(~any( . > 1.0)) %>% 
                  colnames()
   return(fraquezas)
}

find.best.partner.old <- function(nome, legendary = NA) {
  if (check.pokemon(nome)) {
    weakness.pkmn <- find.pokemon.weakness(nome)
    chosen.pkmn <- pokemon[pokemon$name == nome,]
    partner <- pokemon %>% 
                filter(!!as.symbol(weakness.pkmn) < 1.0 &
                        base_total >= (chosen.pkmn$base_total * 0.5) &
                        name != chosen.pkmn$name) %>% # situations like Rayquaza
                arrange(desc(base_total))
    
    if(!is.na(legendary) & !legendary) {
      partner <- partner %>% filter(is_legendary == 0) %>% head(1)
    } else {
      partner <- head(partner, 1)
    }
    return(partner)
  }
}

# achar pkmn que dao > 1.0 em weakness.pkmn
# pkmn que tem type1 ou type2 que tem o maior somatoria de weakness.pkmn
find.best.partner <- function(nome, legendary = NA) {
  if (check.pokemon(nome)) {
    weakness.pkmn <- find.pokemon.weakness(nome)
    weakness.as.sum <- paste(weakness.pkmn, collapse = "+")
    chosen.pkmn <- pokemon[pokemon$name == nome,]
    partner <- pokemon %>% 
                filter(base_total >= (chosen.pkmn$base_total * 0.5) &
                        name != chosen.pkmn$name) %>% # situations like Rayquaza
                mutate(sum.weakness = eval(parse(text=weakness.as.sum)) ) %>% 
                arrange(sum.weakness, desc(base_total)) 
    
    if(!is.na(legendary) & !legendary) {
      partner <- partner %>% filter(is_legendary == 0) %>% head(1)
    } else {
      partner <- head(partner, 1)
    }
    
    # drop the mutated column before returning
    partner <- select(partner, -sum.weakness)
    return(partner)
  }
}


find.best.partner("Aerodactyl", legendary=FALSE) %>% 
  select(name, type1, type2, generation, base_total, abilities) %>% 
  print.datatable.treinador(paste("Um bom parceiro para o Aerodactyl é o", .$name, sep=" "))

```
E se agora tentarmos montar um time inteiro, focando apenas no pokemon informado?
```{r}
target.pkmn.name <- "Aerodactyl"

my.team <- pokemon[pokemon$name == target.pkmn.name, ]
while (nrow(my.team) < 6) {
  last.pkmn.name <- tail(my.team, 1)$name
  next.partner <- find.best.partner(last.pkmn.name, legendary=FALSE)
  pokemon <- pokemon %>% filter(name != last.pkmn.name)
  my.team <- rbind(my.team, next.partner)
}

# run this again to load the dataset
pokemon = read.csv(downloaded.file.path, stringsAsFactors=TRUE, encoding="UTF-8")

my.team %>% 
  select(name, type1, type2, generation, base_total, abilities) %>% 
  print.datatable.treinador("Time gerado")
```

