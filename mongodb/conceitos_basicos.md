## Comandos Básicos do MongoDB


- Característica Básica de uma Operação

[database] . [collection] . [operação]


- Criando uma coleção

```js
db.createCollection("alunos")

```

Podemos tornar os resultados da operação de uma maneira mais agrádavel, utilziando
o comando `.pretty()` no final de cada consulta

- Inserção:


```js
db.alunos.insert({
    "nome": "Geovani",
    "data_nascimento": new Date(1994, 02, 26),
    "notas": [10, 9, 4.5],
    "curso": {
        "nome": "Sistemas de informação"
    },
    "habilidades": [
        {
            "nome": "Inglês",
            "nível": "Avançado"
        }
    ] 
})
```

Além dos dados inseridos o `mongodb` se encarrega de criar um campo chamado `_id` que tem como valor um `ObjectId( valor associado ao id )`

- Busca:

Podemos fazer uma busca geral da seguinte maneira

```js
db.alunos.find()
```

E para filtrar dados podemos passar um objeto:
```js
db.alunos.find({
    "nome": "Geovani"
})
```

Além do find temos o `findOne` que funciona da mesma maneira

- Remoção:

Da mesma maneira podemos fazer uma deleção por um `id`:

```js
db.alunos.remove({
    "_id": ObjectId("123456789")
})
```

## Consultas com OR, AND e IN

Podemos utilizar os `atomics operators` para fazer algumas consultas mais
complexas, utilizando: `$or`, `$and` e `$in`

Exemplos:

```js
db.alunos.find({
    $or : [
        {"curso.nome" : "Sistemas de informação"},
        {"curso.nome" : "Engenharia Química"}    
    ],
    "nome" : "Geovani"
})
```

A mesma consulta pode ser feita utilizando o `in` 

```js
db.alunos.find({
    "curso.nome" : {
        $in : ["Sistema de informação", "Engenharia Química"]
        }
})
```

## Atualizando Dados

Utilizando apenas `update` iremos trocar todo um documento, então é necessário
tomar cuidado para não apagar dados, por exemplo:

```js
db.alunos.update(
    {
        "curso.nome" : "Sistema de informação"
    },
    {
        "nome" : "Sistemas de informação"
    }
)
```

Neste exemplo estamos pegando o primeiro documento com `curso.nome` igual a `Sistemas de Informação` e trocando todo seu conteúdo por ` { nome: "Sistemas de Informação"}

Dessa maneira para mudar um campo precisamos utilizar outro operador atômico, que é o `$set`dessa maneira podemos substituir um valor:


```js
db.alunos.update(
    {"curso.nome" : "Sistema de informação"},
    {
        $set : {
            "curso.nome" : "Sistemas de informação"
        }
    }    
)
```

Dessa maneira conseguimos fazer as operações corretamentes, porém, há ainda uma pequena característica, que é fazer a atualização em apenas um documento, para atualizar todos documentos é necessário especificar que `multi: true`

```js
db.alunos.update(
    {"curso.nome" : "Sistemas de informação"},
    {
        $set : {"curso.nome" : "Sistemas de Informação"}  
    }, 
    {
        multi : true 
    }
)
```

## Push

No caso de array caso seja necessário fazer uma alteração em um vetor podemos adicionar um elemento com o `push`:

```js
db.alunos.update(
    {"_id" : ObjectId("56cb0002b6d75ec12f75d3b5")},
    {
        $push : {
            notas : 8.5
        }    
    }
)
```

Dessa maneira conseguimos inserir um elemento em um array, porém não podemos passar um array diretamente para o `$push`:

```js
$push: {
    notas: [8.5, 9.3, 5.5]
}
```

Dessa maneira iremos inserir um novo array dentro de `notas`, se quisermos que para cada elemento do array fazer a inserção devemos utilizar um outro operador atômico, responsável por fazer essa inserção unitária : `$each`

```js
$push: {
    notas: { 
        $each: [8.5, 9.3, 5.5]
    }
}
```

## Limitando e Ordenando os Dados

Podemos filtrar os dados com alguns operadores atômicos para fazer selecionar os dados desejados, alguns exemplos são:

- `$gt`: Greater Than
- `$lt`: Less Than

Por exemplo:

```js
db.alunos.findOne({
    notas : { $gt : 5}
})
```

Para ordernar temos o método `.sort()` onde podemos selecionar o campo que desejamos como base e os seguintes valores:
- 1 : Ordem Crescente
- -1: Ordem Decrescente

```js
db.alunos.find().sort({
    "nome" : 1
    })
```

Para limitar a quantidade de valores temos um outro operador: `$limit` onde podemos passar a quantidade de elementos desejados:

```js
db.alunos.find(
    {},
{
    $limit: 5
})
```

Da mesma forma podemos pular um elemento com o `$skip` 

## Avançando com  o Mongo: Busca por Proximidades

Muitas vezes podemos querer separar os dados de acordo com a localização, para isso temos algumas palavras especiais como o `coordiantes` que é um array com a `posição x e y`, e o `type` que indica o tipo de coordenada

```js
{
	"nome" : "Geovani",
	"localizacao" : { 
        "type" : "Point",
        "coordinates" : [-23.5882133, -46.63235580000003]
    }
}
```

Dado que tenhamos um conjunto de dados podemos trabalhar com a agregação deles:

``` 
db.alunos.aggregate([
{
    $geoNear : {
        near : {
            coordinates: [-23.5640265, -46.6527128],
            type : "Point"
        }

    }
}
])
```
Novamente temos um operador atômico, dessa vez o `$geoNear` responsável por fazer buscas geográficas, entretanto quando utilizamos uma agregação é necessário indicar para o `mongodb` em qual campo de nossa coleção ele deve utilizar, e para isso precisamos criar também um `index`:

``` 
db.alunos.createIndex({
    localizacao : "2dsphere"
})
```

Com o `$geoNear` é necessário identificar que queremos que os cálculos sejam feitos com base em uma esfera e não uma linha, e que queremos que o resultado seja adicionado em um campo chamado `distancia.calculada` e que desejamos um número ( `num`) especificos de pontos

Dessa forma podemos utilizar todo conhecimento para fazer uma busca completa, como: 

```
db.alunos.aggregate([
{
    $geoNear : {
        near : {
            coordinates: [-23.5640265, -46.6527128],
            type : "Point"
        },
        distanceField : "distancia.calculada",
        spherical : true,
        num : 4
    }
},
{ $skip :1 }
])
```