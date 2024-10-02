Hoje iremos criar um projeto simples utilizando express + prisma.

# Inicialização do projeto
Para iniciar você pode escolher uma pasta, nesse caso iremos chamar de prisma-express
```
npm init
```
Irá perguntar o nome da pasta e algumas informações, ler com calma e preencher.

# Configuração do prisma
Nesta configuração iremos acrescentar o prisma no projeto.
```
npx prisma init
```

Esse comando irá criar uma pasta chamada `prisma` que irá conter o arquivo `schema.prisma`, ele serve para nós criamos o schema do nosso banco de dados, em seguida podes adicionar sua schema, irá criar um arquivo `.env`, responsável por guardar a url do banco de dados a ser entregue ao prisma datasource.

```
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "mysql"
  url      = env("DATABASE_URL")
}

model product {
  id Int @id @default(autoincrement())
  name String
  price Int
  createdAt DateTime @default(now())
  
  @@map("products")
}

```
Calma :hand: irei explicar cada bloco de código.
- gerador cliente -> Serve para especificar o cliente de geração do banco, você pode escolher outros, mas iremos usar o padrão do prisma
- datasource db -> define o caminho do banco de dados, lembrando que o prisma é multibanco, então pode apontar em sua .env, qualquer banco relacional facilmente, veja que o provider é o postgresql, e a url chama uma variável de ambiente.
- Em model product, perceba que usamos alguns parâmetros como id, name, price, created_at para sabermos a data em que o produto foi criado.
- Obs: O trecho de código `@@map("products")` serve para específicar o nome da tabela, como ficará gravado no banco de dados.

# Iniciando prisma
```
npx prisma generate
```

# Configuração do Express
Antes de explicar como funciona no express iremos instalar as dependências, é importante saber que você precisa do [nodejs](https://nodejs.org/en) instalado, digite:
```
 npm install -D nodemon
```
Devdependencies, ou sejaseja apenas serão utilizados em modo de desenvolvimento.

Agora instale:
```
npm install dotenv express
```
Esses pacotes são fundamentais para nossa API funciona, em desenvolvimento e produção, agora crie um script no `package.json`
```
"scripts": {
    "dev": "nodemon index",
    "start": "node index.js"
  },
```
Ficará desta forma, iremos utilizar o comando `npm start` para iniciar a API em produção, e `npm run dev` para iniciar em teste.

Feito tudo isso, pode criar um arquivo index.js na base do seu projeto, e adicionar o trecho de código abaixo.
```
import express from 'express'
import cors from 'cors'
import dotenv from 'dotenv'
import ProductRoute from './routes/ProductRoute.js'
dotenv.config()

const app = express()
const port = process.env.APP_PORT || 5000

app.use(cors())
app.use(express.json())
app.get('/', (req, res) => {
    res.send('Hello World!')
})
app.use(ProductRoute)

app.listen(port, () => {
    console.log(`Server listening on port ${port}`)
})

export default app
```

# Configuração das rotas e controllers
Crie uma pasta chamada `routes` e crie um arquivo chamado `ProductRoute.js` dentro dela e adicione seguinte código.
```
import express from 'express'
import { getProducts, getProductById, createProduct, updateProduct, deleteProduct } from '../controller/ProductController.js'

const router = express.Router()

router.get('/products', getProducts)
router.get('/products/:id', getProductById)
router.post('/products', createProduct)
router.patch('/products/:id', updateProduct)
router.delete('/products/:id', deleteProduct)

export default router
```

Crie uma pasta chamada `controller`, em seguida, adicione um arquivo chamado `ProductController.js` adicione o seguinte código.
```
import { PrismaClient } from '@prisma/client'
const prisma = new PrismaClient()

export const getProducts = async (req, res) => {
    try {
        const response = await prisma.product.findMany()
        res.status(200).json(response)
    } catch (error) {
        res.status(500).json({ msg: error.message })
    }
}

export const getProductById = async (req, res) => {
    try {
        const response = await prisma.product.findUnique({
            where: {
                id: Number(req.params.id),
            },
        })
        res.status(200).json(response)
    } catch (error) {
        res.status(404).json({ msg: error.message })
    }
}

export const createProduct = async (req, res) => {
    const { name, price } = req.body
    try {
        const product = await prisma.product.create({
            data: {
                name: name,
                price: price,
            },
        })
        res.status(201).json(product)
    } catch (error) {
        res.status(400).json({ msg: error.message })
    }
}

export const updateProduct = async (req, res) => {
    const { name, price } = req.body
    try {
        const product = await prisma.product.update({
            where: {
                id: Number(req.params.id),
            },
            data: {
                name: name,
                price: price,
            },
        })
        res.status(200).json(product)
    } catch (error) {
        res.status(400).json({ msg: error.message })
    }
}

export const deleteProduct = async (req, res) => {
    try {
        const product = await prisma.product.delete({
            where: {
                id: Number(req.params.id),
            },
        })
        res.status(200).json(product)
    } catch (error) {
        res.status(400).json({ msg: error.message })
    }
}
```

Uma vez que você tenha um banco postgresql, que pode ser criado com 1 único click usando o servidor da [railway](https://railway.app/), copie e cole a url do banco no arquivo .env, substituindo a que está lá.

ficaria assim:
```
DATABASE_URL="mysql://root:123456@localhost:3306/prisma-react"
```

# Criando migração e testando API
Agora iremos criar essas tabelas no banco e testar, para isso rode o comando:
```
npx prisma migrate dev --name first-migrate
```

Se você receber está mensagem em verde no seu terminal, deu tudo certo.

```
Your database is now in sync with your schema.
```

Apenas rode o comando `npm run dev` e pode testar sua API a vontade, feita em prisma, utilize [Insomnia](https://insomnia.rest/download) para realizar este teste.

