# Desenvolvimento de uma aplicação Serverless na AWS

## Arquitetura da aplicação

![Intro](./imgs/intro.png)

- Serviços utilizados da AWS:
1. API Gateway
2. Lambda
3. DynamoDB

- Criação de um CRUD simples.
1. Listar um único registro
2. Listar todos os registros
3. Criar registro
4. Editar registro
5. Excluir registro

## 1) Criação do usuário

Como eu já tenho uma conta na AWS, irei criar um usuário no IAM para a realização do trabalho.

- Acessar o painel do IAM e clicar em **Users**.

![Imagem 1](./imgs/1.png)

- Clicar em Create **User**.

![Imagem 2](./imgs/2.png)

- Colocar um nome, dar acesso ao AWS Management Console, pois irei desenvolver o trabalho no console, e, por facilidade, escolher a opção **I want to create an IAM user**.

- Irá aparecer opções de gerar a senha do usuário automaticamente, ou criar uma nova senha, com a opção do usuário criar uma nova senha no próximo login. 

- Como serei eu que irei usar esse usuário, desabilitei essa opção de criar nova senha no próximo login. Clicar em **Next** para prosseguir com a criação do usuário.

![Imagem 3](./imgs/3.png)

- Para as permissões, por facilidade, dei acesso de administrador (**AdministratorAccess**) para o usuário. Clicar em **Next**.

![Imagem 4](./imgs/4.png)

- Revisar as opções escolhidas, e criar uma tag, se necessário. Se estiver tudo certo, clicar em **Create User**.

![Imagem 5](./imgs/5.png)

- Aparecerá essa tela para pegar as informações da url de login, o nome do usuário e a senha gerada pela AWS. Pode copiar essas informações e salvar em um documento, ou clicar em **Download .csv file** para baixar as informações em um arquivo .csv. Essa tela só aparecerá dessa vez.

![Imagem 6](./imgs/6.png)

- Acessar o link de login (nesse caso, https://132584256848.signin.aws.amazon.com/console), e colocar o IAM user name e a senha definidas anteriormente. No momento em que acessar esse link, o console do root será deslogado.

![Imagem 7](./imgs/7.png)
![Imagem 8](./imgs/8.png)

- Com isso, podemos começar a criar os serviços a partir deste usuário.

![Imagem 9](./imgs/9.png)

## 2) Criação do banco de dados no DynamoDB

- Para criar o banco de dados no DynamoDB, basta procurar por DynamoDB na busca e clicar no serviço.

![Imagem 10](./imgs/10.png)

- Clicar em **Create table**.

![Imagem 11](./imgs/11.png)

- Em **Table name**, colocar o nome da tabela (*todos*) e, em **Partition Key**, colocar o nome *id* e o tipo da variável como *Number*. Deixar as outras opções como está e clicar em **Create table**.

![Imagem 12](./imgs/12.png)
![Imagem 13](./imgs/13.png)

- Esperar até a tabela ser criada com sucesso.

![Imagem 14](./imgs/14.png)
![Imagem 15](./imgs/15.png)

## 3) Criação e teste das funções de criação de dados no Lambda

- Para acessar o serviço do Lambda, basta procurar por Lambda na barra de busca e clicar no serviço.

![Imagem 16](./imgs/16.png)

A primeira função a ser criada será o método POST do CRUD.
- Clicar em **Create function**.
- Escolher a opção Author from scratch, que irá criar um exemplo de Hello World. 
- Em **Function name**, colocar o nome da função (2mobr-devops-trabalho-final-post) 
- Escolher a **Runtime** da Lambda. No momento de criação deste trabalho, a runtime do Node.js 14.x não está mais disponível (que foi usada e recomendada no momento da gravação da aula). Por conta disso, vou escolher o Node.js 20.x.
- Deixar as outras opções sem alteração e clicar em **Create function**.

![Imagem 17](./imgs/17.png)
![Imagem 18](./imgs/18.png)

- Vamos criar duas variáveis de ambiente. A primeira é o **DEBUG**, que, caso o valor dela seja verdadeiro, serve para dar um print de informações no console, e também guardar um log dessas informações no CloudWatch. e a segunda variável é o **TABLE**, que será o nome da tabela do DynamoDB, (todos).
- Clicar na aba **Configuration** e, no lado esquerdo, clicar em **Environment variables**. Em seguida, clicar em **Edit** para adicionar as variáveis.

![Imagem 19](./imgs/19.png)

- Clicar em **Add environment variable** e, nos campos que apareceram, colocar em **Key** o valor **DEBUG**, e o **Value** com o valor **true**. Clicar novamente em **Add environment variable** e, em **Key**, colocar **TABLE**, e em **Value**, colocar todos. Clicar em **Save**.

![Imagem 20](./imgs/20.png)

- Voltando para a aba **Code**, vamos criar dois arquivos. O primeiro é o **normalizer.mjs**, que irá filtrar as informações recebidas do API Gateway na variável *event*, e devolver para o handler o método, o corpo da requisição, a querystring e os path parameters. Para criar o arquivo, clicar com o botão direito no mouse na pasta e clicar em **New File**, e nomear esse novo arquivo como normalizer.mjs.

![Imagem 21](./imgs/21.png)

- Colocar o código fonte abaixo e clicar em **Deploy** para salvar o arquivo.

```typescript
export const normalizeEvent = event => {
    const data = (typeof event['body'] === 'string' || event['body'] instanceof String) 
        ? JSON.parse(event['body']) 
        : event['body'];
    
    return {
        method: event['requestContext']['http']['method'],
        data: data || {},
        querystring: event['queryStringParameters'] || {},
        pathParameters: event['pathParameters'] || {},
    };
}
```

- Um ponto a mais que adicionei neste código foi a verificação se o corpo é uma string, faz a conversão para JSON, senão, é já é um JSON e devolve do jeito que está.

![Imagem 22](./imgs/22.png)

- O segundo arquivo é o **response.mjs**, que padroniza a resposta a ser enviada para a API Gateway. Para criar o arquivo, clicar com o botão direito no mouse na pasta e clicar em **New File**, e nomear esse novo arquivo como **response.mjs**.

![Imagem 23](./imgs/23.png)

- Colocar o código fonte abaixo e clicar em **Deploy** para salvar o arquivo.

```typescript
export const response = (status, body) => {
    return {
        statusCode: status,
        body: JSON.stringify(body),
        headers: {
            'Content-Type': 'application/JSON'
        }
    }
}
```

![Imagem 24](./imgs/24.png)

- Agora vamos criar a função que salva as informações no DynamoDB, mas primeiro precisamos dar permissão à função para se comunicar com o banco. Para isso, na aba **Configuration**, escolha a opção **Permissions** no lado esquerdo, e clicar no link do **Role name**, que irá abrir uma nova aba do navegador.

![Imagem 25](./imgs/25.png)

- Em **Permissions policies**, clicar no link do **Policy name**, e irá abrir uma outra aba.

![Imagem 26](./imgs/26.png)

- Clicar em **Edit**

![Imagem 27](./imgs/27.png)

- Clicar em **Visual** para facilitar a edição, clicar em **Add more permissions**, procurar por **DynamoDB** e clicar em **DynamoDB**.

![Imagem 28](./imgs/28.png)

- Por facilidade, clicar em **All DynamoDB Actions (dynamodb:*)** e, em **Resources**, escolher a opção **All**. O ideal seria escolher os *Access Levels* e os *Resources* adequados para a operação de escrita. Clicar em **Next**.

![Imagem 29](./imgs/29.png)

- Clicar em **Save changes** e verificar se a política do DynamoDB foi criada.

![Imagem 30](./imgs/30.png)
![Imagem 31](./imgs/31.png)

- Agora podemos fazer a função que salva os dados no DynamoDB. Volte para a página do *Lambda* e clique na aba *Code*. Como ainda não temos o API Gateway, podemos fazer um teste com o próprio recurso do Lambda. Para isso, clicar na setinha ao lado do *Test*, e escolher *Configure test event*. Abrirá um modal. 

![Imagem 32](./imgs/32.png)

- Em colocar um nome no **Event name**, e em **Template**, escolher **apigateway-http-api-proxy**, que é o payload que o API Gateway irá enviar para a função. Colocar no body um JSON parecido com o do print, com *id* sendo *number*, *task* sendo *string* e *done* sendo *boolean*. Clique em **Save**.

![Imagem 33](./imgs/33.png)

- No arquivo **index.mjs**, colocar o seguinte código fonte e clicar em **Deploy**:

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { PutCommand, DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

import { normalizeEvent } from './normalizer.mjs';
import { response } from './response.mjs';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  if (process.env.DEBUG) {
      console.log({
          message: 'Received event',
          data: JSON.stringify(event),
      });
  }
  
  const table = event.table || process.env.TABLE;
  if (!table) {
      throw new Error('No table name defined.');
  }

  const { data } = normalizeEvent(event);
  
  const command = new PutCommand({
      TableName: table,
      Item: {
          ...data,
          created_at: new Date().toISOString(),
      },
  });
    
  try {
      await docClient.send(command);

      console.log({
          message: 'Record has been created',
          data: JSON.stringify(command),
      });

      return response(201, `Record ${data.id} has been created`);
  } catch (err) {
      console.error(err);
      return response(500, { err, data });
  }
};

```

- Por conta da runtime do Lambda ser Node.js 20.x, o código é um pouco diferente do exemplo, já que essa runtime usa a v3 do SDK, mas o conceito é o mesmo: 
    1. instancia o DocumentClient do DynamoDB;
    2. se tiver o DEBUG ativo, manda um log pro CloudWatch que o Lambda recebeu o evento;
    3. se não tiver o TABLE, estoura um erro;
    4. faz a normalização dos dados;
    5. cria o comando para inserir os dados na tabela do DynamoDB;
    6. Tenta enviar os dados para o DynamoDB, se der certo envia a resposta com status code 201, se não, envia a resposta com status code 500.

- Clicar em **Test** para ver se funciona.

![Imagem 34](./imgs/34.png)

- Se funcionou, ir ao DynamoDB para verificar se gravou no banco. Para isso, abrir o console do **DynamoDB**. No lado esquerdo, clicar em **Tables**.

![Imagem 35](./imgs/35.png)

- Clicar em **todos**.

![Imagem 36](./imgs/36.png)

- Clicar em **Explore table items**.

![Imagem 37](./imgs/37.png)

- Verificar se o item foi criado.

![Imagem 38](./imgs/38.png)

- Também podemos ver se os logs foram gravados no CloudWatch. Ir no console do **CloudWatch** e, no lado esquerdo, clicar em **Log groups**.

![Imagem 39](./imgs/39.png)

- Clicar no item relativo ao lambda do post (nesse caso o **/aws/lambda/2mobr-devops-trabalho-final-post**)

![Imagem 40](./imgs/40.png)

- Na parte de **Log streams**, procurar o log relacionado.

![Imagem 41](./imgs/41.png)
![Imagem 42](./imgs/42.png)

## 4) Criação e teste da API Gateway e do endpoint POST, integrando com a função do Lambda

- Para acessar o API Gateway, basta buscar por **API Gateway** na barra de busca e clicar no serviço.

![Imagem 43](./imgs/43.png)

- Escolher **HTTP API** e clicar em **Build**.

![Imagem 44](./imgs/44.png)

- Em **API name**, escolher um nome e clicar em **Next**.

![Imagem 45](./imgs/45.png)

- Como vamos configurar as rotas depois, clicar em **Next**.

![Imagem 46](./imgs/46.png)

- Não vamos precisar criar stages de deploy, já que a AWS cria um por padrão e o deploy é feito automaticamente, clicar em **Next**.

![Imagem 47](./imgs/47.png)

- Revisar e clicar em **Create**.

![Imagem 48](./imgs/48.png)

- Em **Routes**, clicar em **Create**.

![Imagem 49](./imgs/49.png)

- Em **Route and method**, escolher **POST** e colocar a rota como **/v1/todos**. Clicar em **Create**.

![Imagem 50](./imgs/50.png)

- Escolher o **POST** e clicar em **Attach integration**.

![Imagem 51](./imgs/51.png)

- Clicar em **Create and attach an integration**.

![Imagem 52](./imgs/52.png)

- Em **Integration type**, escolher **Lambda function**.

![Imagem 53](./imgs/53.png)

- Em **Lambda function**, escolher a função relacionada ao POST e clicar em **Create**.

![Imagem 54](./imgs/54.png)

- Com isso, o endpoint do POST estará associado ao Lambda criado anteriormente.

![Imagem 55](./imgs/55.png)
![Imagem 56](./imgs/56.png)

- Voltar para o API Gateway e, em **Stages**, copiar a **Invoke URL**.

![Imagem 57](./imgs/57.png)

- Abrir o Postman e colocar o Invoke URL, seguido de /v1/todos, que foi o endpoint criado anteriormente. Mudar o método para **POST**, e, em **Body**, selecionar raw no select do lado esquerdo e **JSON** no select do lado direito. No corpo da mensagem, colocar um JSON parecido com o feito no teste do Lambda, e clicar em **Send**.

```json
{
    "id": 2,
    "task": "Teste do lambda 2",
    "done": false
}

```

![Imagem 58](./imgs/58.png)

- Verificar no DynamoDB se o registro foi salvo.

![Imagem 59](./imgs/59.png)

## 5) Criação e teste das demais funções do CRUD no Lambda, dos demais endpoints no API Gateway

### Método GET - listagem única de item

- Criação do Lambda

![Imagem 60](./imgs/60.png)

- Permissão do DynamoDB

![Imagem 61](./imgs/61.png)

- Variáveis de ambiente do Lambda

![Imagem 62](./imgs/62.png)

- Código fonte do index.mjs

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { GetCommand , DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

import { normalizeEvent } from './normalizer.mjs';
import { response } from './response.mjs';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  if (process.env.DEBUG) {
      console.log({
          message: 'Received event',
          data: JSON.stringify(event),
      });
  }
  
  const table = event.table || process.env.TABLE;
  if (!table) {
      throw new Error('No table name defined.');
  }

  const { pathParameters } = normalizeEvent(event);
  
  const command = new GetCommand({
      TableName: table,
      Key: {
        id: parseInt(pathParameters['todoId'], 10)
      }
  });
    
  try {
      const data = await docClient.send(command);

      console.log({
          message: 'Records found',
          data,
      });
      
      if (data.Item) {
        return response(200, data.Item);
      } else {
        return response(404, 'Item not found');
      }

  } catch (err) {
      console.error(err);
      return response(500, { err });
  }
};

```
1) instancia o DocumentClient do DynamoDB;
2) se tiver o DEBUG ativo, manda um log pro CloudWatch que o Lambda recebeu o evento;
3) se não tiver o TABLE, estoura um erro;
4) pega o parâmetro do caminho a partir do normalizer;
5) cria o comando para buscar o dado na tabela do DynamoDB a partir do parâmetro;
6) Tenta buscar os dados para o DynamoDB. Se der certo e tiver dado, envia a resposta com status code 200 e o registro encontrado, se não, envia a resposta com status 404. Em caso de erro, envia a resposta com status code 500.

- Código fonte do normalizer.mjs (mesma coisa do POST)

```typescript
export const normalizeEvent = event => {

    const data = (typeof event['body'] === 'string' || event['body'] instanceof String) 
        ? JSON.parse(event['body']) 
        : event['body'];
    
    return {
        method: event['requestContext']['http']['method'],
        data: data || {},
        querystring: event['queryStringParameters'] || {},
        pathParameters: event['pathParameters'] || {},
    };
}
```

- Código fonte do response.mjs (mesma coisa do POST)

```typescript
export const response = (status, body) => {
    return {
        statusCode: status,
        body: JSON.stringify(body),
        headers: {
            'Content-Type': 'application/json',
        },
    };
};
```
- Criação do teste do Lambda

![Imagem 63](./imgs/63.png)

- Criação do endpoint no API Gateway e integração com o Lambda

   Rota: /v1/todos/{todoId} GET

![Imagem 64](./imgs/64.png)

- Teste no Postman

![Imagem 65](./imgs/65.png)
![Imagem 66](./imgs/66.png)

### Método GET - listagem de todos os items

- Criação do Lambda

![Imagem 67](./imgs/67.png)

- Permissão do DynamoDB

![Imagem 68](./imgs/68.png)

- Variáveis de ambiente do Lambda

![Imagem 69](./imgs/69.png)

- Código fonte do index.mjs

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { ScanCommand  , DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

import { response } from './response.mjs';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  if (process.env.DEBUG) {
      console.log({
          message: 'Received event',
          data: JSON.stringify(event),
      });
  }
  
  const table = event.table || process.env.TABLE;
  if (!table) {
      throw new Error('No table name defined.');
  }
  
  const command = new ScanCommand ({
      TableName: table,
  });
    
  try {
      const data = await docClient.send(command);

      console.log({
          message: 'Records found',
          data,
      });
      
      if (data.Items) {
        return response(200, data.Items);
      } else {
        return response(404, 'Item not found');
      }

  } catch (err) {
      console.error(err);
      return response(500, { err });
  }
};
```
1. instancia o DocumentClient do DynamoDB;
2. se tiver o DEBUG ativo, manda um log pro CloudWatch que o Lambda recebeu o evento;
3. se não tiver o TABLE, estoura um erro;
4. pega o parâmetro do caminho a partir do normalizer;
5. cria o comando para escanear a tabela do DynamoDB;
6. Tenta buscar os dados para o DynamoDB. Se der certo e tiver dado, envia a resposta com status code 200 e os registros encontrados, se não, envia a resposta com status 404. Em caso de erro, envia a resposta com status code 500.

- Nessa função não precisou do normalizer.mjs

- Código fonte do response.mjs (mesma coisa do POST)

```typescript
export const response = (status, body) => {
    return {
        statusCode: status,
        body: JSON.stringify(body),
        headers: {
            'Content-Type': 'application/json',
        },
    };
};
```

- Criação do teste no Lambda

![Imagem 70](./imgs/70.png)
![Imagem 71](./imgs/71.png)

- Criação do endpoint no API Gateway e integração com o Lambda
    
    Rota: /v1/todos GET

![Imagem 72](./imgs/72.png)

- Teste no Postman

![Imagem 73](./imgs/73.png)

### Método PUT

- Criação do Lambda

![Imagem 74](./imgs/74.png)

- Permissão do DynamoDB

![Imagem 75](./imgs/75.png)

- Variáveis de ambiente do Lambda

![Imagem 76](./imgs/76.png)

- Código fonte do index.mjs

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { UpdateCommand, GetCommand  , DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

import { normalizeEvent } from './normalizer.mjs';
import { response } from './response.mjs';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  if (process.env.DEBUG) {
      console.log({
          message: 'Received event',
          data: JSON.stringify(event),
      });
  }
  
  const table = event.table || process.env.TABLE;
  if (!table) {
      throw new Error('No table name defined.');
  }
  
  const { data } = normalizeEvent(event);
  
  const getCommand = new GetCommand({
      TableName: table,
      Key: {
        id: parseInt(data['id'], 10)
      }
  });

  try {
    const data = await docClient.send(getCommand);
    
    console.log({ data });
    
    if (!data.Item) {
      return response(404, 'Item not found');
    }
  } catch (err) {
    console.error(err);
    return response(500, { err });
  }
  
  
  const command = new UpdateCommand ({
      TableName: table,
      Key: {
        id: parseInt(data['id'], 10)
      },
      UpdateExpression: "set done = :done, created_at = :created_at",
      ExpressionAttributeValues: {
        ":done": data.done,
        ":created_at": new Date().toISOString(),
      },
      ReturnValues: "ALL_NEW",
  });
    
  try {
      const item = await docClient.send(command);

      console.log({
          message: 'Record has been updated',
          item,
      });
      
      return response(200, `Record ${data.id} has been updated`);

  } catch (err) {
      console.error(err);
      return response(500, { err });
  }
};
```

1. Instancia o DocumentClient do DynamoDB;
2. Se tiver o DEBUG ativo, manda um log pro CloudWatch que o Lambda recebeu o evento;
3. Se não tiver o TABLE, estoura um erro;
4. Faz uma busca na tabela com o id escolhido. Se não tiver, responde com status code 404;
5. Cria o comando para buscar o dado na tabela do DynamoDB a partir do parâmetro;
6. Tenta enviar os dados para editar para o DynamoDB. Se der certo e tiver dado, envia a resposta com status code 200 e a mensagem que o registro foi atualizado. Em caso de erro, envia a resposta com status code 500.

- Código fonte do normalizer.mjs (mesma coisa do POST)

```typescript
export const normalizeEvent = event => {

    const data = (typeof event['body'] === 'string' || event['body'] instanceof String) 
        ? JSON.parse(event['body']) 
        : event['body'];
    
    return {
        method: event['requestContext']['http']['method'],
        data: data || {},
        querystring: event['queryStringParameters'] || {},
        pathParameters: event['pathParameters'] || {},
    };
}
```

- Código fonte do response.mjs (mesma coisa do POST)

```typescript
export const response = (status, body) => {
    return {
        statusCode: status,
        body: JSON.stringify(body),
        headers: {
            'Content-Type': 'application/json',
        },
    };
};
```

- Criação do teste do Lambda

![Imagem 77](./imgs/77.png)
![Imagem 78](./imgs/78.png)
![Imagem 79](./imgs/79.png)

- Criação do endpoint no API Gateway e integração com o Lambda

    Rota: /v1/todos PUT

![Imagem 80](./imgs/80.png)

- Teste no Postman

![Imagem 81](./imgs/81.png)
![Imagem 82](./imgs/82.png)
![Imagem 83](./imgs/83.png)

### Método DELETE

- Criação do Lambda

![Imagem 84](./imgs/84.png)

- Permissão do DynamoDB

![Imagem 85](./imgs/85.png)

- Variáveis de ambiente do Lambda

![Imagem 86](./imgs/86.png)

- Código fonte do index.mjs

```typescript
import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DeleteCommand, GetCommand, DynamoDBDocumentClient } from "@aws-sdk/lib-dynamodb";

import { normalizeEvent } from './normalizer.mjs';
import { response } from './response.mjs';

const client = new DynamoDBClient({});
const docClient = DynamoDBDocumentClient.from(client);

export const handler = async (event) => {
  if (process.env.DEBUG) {
      console.log({
          message: 'Received event',
          data: JSON.stringify(event),
      });
  }
  
  const table = event.table || process.env.TABLE;
  if (!table) {
      throw new Error('No table name defined.');
  }
  
  const { data } = normalizeEvent(event);
  
  const getCommand = new GetCommand({
      TableName: table,
      Key: {
        id: parseInt(data['id'], 10)
      }
  });

  try {
    const data = await docClient.send(getCommand);
    
    console.log({ data });
    
    if (!data.Item) {
      return response(404, 'Item not found');
    }
  } catch (err) {
    console.error(err);
    return response(500, { err });
  }
  
  
  const command = new DeleteCommand ({
      TableName: table,
      Key: {
        id: parseInt(data['id'], 10)
      }
  });
    
  try {
      const item = await docClient.send(command);

      console.log({
          message: 'Record has been deleted',
          item,
      });
      
      return response(200, `Record ${data.id} has been deleted`);

  } catch (err) {
      console.error(err);
      return response(500, { err });
  }
};
```

1. Instancia o DocumentClient do DynamoDB;
2. Se tiver o DEBUG ativo, manda um log pro CloudWatch que o Lambda recebeu o evento;
3. Se não tiver o TABLE, estoura um erro;
4. Faz uma busca na tabela com o id escolhido. Se não tiver, responde com status code 404;
5. Cria o comando para excluir o dado na tabela do DynamoDB a partir do parâmetro;
6. Tenta enviar os dados para exclusão para o DynamoDB. Se der certo e tiver dado, envia a resposta com status code 200 e a mensagem que o registro foi apagado. Em caso de erro, envia a resposta com status code 500.

- Código fonte do normalizer.mjs (mesma coisa do POST)

```typescript
export const normalizeEvent = event => {

    const data = (typeof event['body'] === 'string' || event['body'] instanceof String) 
        ? JSON.parse(event['body']) 
        : event['body'];
    
    return {
        method: event['requestContext']['http']['method'],
        data: data || {},
        querystring: event['queryStringParameters'] || {},
        pathParameters: event['pathParameters'] || {},
    };
}
```

- Código fonte do response.mjs (mesma coisa do POST)

```typescript
export const response = (status, body) => {
    return {
        statusCode: status,
        body: JSON.stringify(body),
        headers: {
            'Content-Type': 'application/json',
        },
    };
};
```

- Criação do teste do Lambda

![Imagem 87](./imgs/87.png)
![Imagem 88](./imgs/88.png)
![Imagem 89](./imgs/89.png)
![Imagem 90](./imgs/90.png)

- Criação do endpoint no API Gateway e integração com o Lambda

    Rota: /v1/todos DELETE

![Imagem 91](./imgs/91.png)

- Teste no Postman

![Imagem 92](./imgs/92.png)
![Imagem 93](./imgs/93.png)
![Imagem 94](./imgs/94.png)
