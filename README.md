# dynamodb-dio



#### Criando uma  tabela Book
A tabela terá o atributo **Author** como **primary key** e o atributo **BookTitle** como **sort key** 

```
aws dynamodb create-table \
    --table-name Book \
    --attribute-definitions \
        AttributeName=Author,AttributeType=S \
        AttributeName=BookTitle,AttributeType=S \
    --key-schema \
        AttributeName=Author,KeyType=HASH \
        AttributeName=BookTitle,KeyType=RANGE \        
    --provisioned-throughput \
        ReadCapacityUnits=5,WriteCapacityUnits=5 \
    --table-class STANDARD
```

#### Inserindo itens na tabela Book
Com a tabela criada, foi inserido quatro itens.

```
aws dynamodb put-item \
    --table-name Book \
    --item \
        '{"Author" : {"S" : "Christopher Paolini" }, "BookTitle" : {"S" : "Eragon"},"Year" : {"N" : 2005} }'

aws dynamodb put-item \
    --table-name Book \
    --item \
        '{"Author" : {"S" : "J K Rowling" }, "BookTitle" : {"S" : "Harry Potter e o Prisioneiro de Azkaban"},"Year" : {"N" : 2000} }'

aws dynamodb put-item \
    --table-name Book \
    --item \
        '{"Author" : {"S" : "John Piper" }, "BookTitle" : {"S" : "Em Busca de Deus"},"Year" : {"N" : 2008} }'

aws dynamodb put-item \
    --table-name Book \
    --item \
        '{"Author" : {"S" : "George Orwell" }, "BookTitle" : {"S" : "A revolução dos Bichos"},"Year" : {"N" : 2007} }'

```
Caso queira salvar vários itens em um único comando, você pode passar um arquivo, e este arquivo deve conter os itens

```
aws dynamodb batch-write-item \
    --request-items file://books.json
```

#### Consultando itens da tabela utilizando as keys
Agora que a tabela possui itens, podemos fazer consultas, nessa consulta foi utilizada a primary key **Author**
Essa query irá retornar todos os livros do autor específicado

```
aws dynamodb query \
    --table-name Book \
    --key-condition-expression "Author = :name" \
    --expression-attribute-values  '{":name":{"S":"Christopher Paolini"}}'
```

#### Consultando itens da tabela sem utilizar as keys
Para fazer uma consulta passando o atributo **Year**, temos que criar um **secondary index**

```
aws dynamodb update-table \
    --table-name Book \
    --attribute-definitions AttributeName=Year,AttributeType=N \
    --global-secondary-index-updates \
        "[{\"Create\":{\"IndexName\": \"Year-index\",\"KeySchema\":[{\"AttributeName\":\"Year\",\"KeyType\":\"HASH\"}], \
        \"ProvisionedThroughput\": {\"ReadCapacityUnits\": 10, \"WriteCapacityUnits\": 5      },\"Projection\":{\"ProjectionType\":\"ALL\"}}}]"
```
Com o index criado, podemos consultar através do atributo **Year** que irá retornar os livros de 2008
```
aws dynamodb query \
    --table-name Book \
    --index-name Year-index \
    --key-condition-expression "Year = :year" \
    --expression-attribute-values  '{":year":{"N": 2008}}'
```


#### Referências
  [Conceitos básicos do DynamoDB](https://docs.aws.amazon.com/pt_br/amazondynamodb/latest/developerguide/GettingStartedDynamoDB.html)
