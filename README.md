# Desafio Dio - Criando um Ebook com ChatGPT &amp; MidJourney



### **Projeto abrangente com muitos códigos para criar um e-book usando o ChatGPT e o MidJourney:**



Neste projeto, construiremos uma aplicação full-stack que usa o ChatGPT e o MidJourney para criar e publicar e-books. A aplicação será construída usando Node.js, React, MongoDB e as APIs do ChatGPT e do MidJourney.



### **Pré-requisitos**

- Conta da OpenAI

- Conta do MidJourney

- Node.js 14 ou superior

- npm 6 ou superior

- MongoDB

  

### **Configuração**

### **Servidor**



1. #### Crie um novo projeto Node.js:

   plaintext

   

   ```plaintext
   npm init -y
   ```

   

2. #### Instale as seguintes dependências:

   plaintext

   

   ```plaintext
   npm install express body-parser mongoose fetch multer socket.io
   ```

   

3. #### Crie um arquivo `server.js` com o seguinte código:

   javascript

   

   ```javascript
   const express = require('express')
   const bodyParser = require('body-parser')
   const mongoose = require('mongoose')
   const fetch = require('node-fetch')
   const multer = require('multer')
   const socketIO = require('socket.io')
   
   const app = express()
   
   app.use(bodyParser.json())
   
   mongoose.connect('mongodb://localhost:27017/ebooks', {
     useNewUrlParser: true,
     useUnifiedTopology: true
   })
   
   const EbookSchema = new mongoose.Schema({
     title: String,
     body: String,
     author: String,
     image: String
   })
   
   const Ebook = mongoose.model('Ebook', EbookSchema)
   
   const storage = multer.diskStorage({
     destination: './uploads/',
     filename: (req, file, cb) => {
       cb(null, file.originalname)
     }
   })
   
   const upload = multer({ storage })
   
   app.post('/ebook', upload.single('image'), async (req, res) => {
     const { prompt, author } = req.body
   
     const response = await fetch('https://generativelanguage.googleapis.com/v1beta2/models/chat-bison-001:generateMessage?key={{API_KEY}}', {
       method: 'POST',
       headers: {
         'Content-Type': 'application/json'
       },
       body: JSON.stringify({
         input: {
           text: prompt
         }
       })
     })
   
     const data = await response.json()
   
     const ebook = {
       title: data.candidates[0].content,
       body: data.candidates[1].content,
       author: author,
       image: req.file.originalname
     }
   
     const midjourneyResponse = await fetch('https://api.midjourney.com/v4/images', {
       method: 'POST',
       headers: {
         'Content-Type': 'application/json',
         'Authorization': `Bearer {{API_KEY}}`
       },
       body: JSON.stringify({
         prompt: {
           text: ebook.title
         }
       })
     })
   
     const midjourneyData = await midjourneyResponse.json()
   
     ebook.image = midjourneyData.data[0].url
   
     const newEbook = new Ebook(ebook)
   
     newEbook.save()
   
     res.json(newEbook)
   })
   
   app.get('/ebooks', async (req, res) => {
     const ebooks = await Ebook.find()
   
     res.json(ebooks)
   })
   
   const io = socketIO(server)
   
   io.on('connection', (socket) => {
     console.log('Novo usuário conectado')
   
     socket.on('new ebook', (ebook) => {
       io.emit('new ebook', ebook)
     })
   
     socket.on('disconnect', () => {
       console.log('Usuário desconectado')
     })
   })
   
   app.listen(3000, () => {
     console.log('Servidor rodando na porta 3000')
   })
   ```



### **Cliente**



1. #### Crie um novo projeto React:

   plaintext

   

   ```plaintext
   npx create-react-app ebooks
   ```

   

2. #### No diretório do projeto React, instale as dependências `axios`, `socket.io-client` e `react-dropzone`:

   plaintext

   

   ```plaintext
   npm install axios socket.io-client react-dropzone
   ```

   

3. #### No arquivo `App.js`, substitua o código existente pelo seguinte:

   javascript

   

   ```javascript
   import React, { useState, useEffect } from 'react'
   import axios from 'axios'
   import socketIOClient from 'socket.io-client'
   import Dropzone from 'react-dropzone'
   
   const App = () => {
     const [prompt, setPrompt] = useState('')
     const [author, setAuthor] = useState('')
     const [ebooks, setEbooks] = useState([])
     const [image, setImage] = useState(null)
   
     useEffect(() => {
       const socket = socketIOClient('http://localhost:3000')
   
       socket.on('new ebook', (ebook) => {
         setEbooks([ebook, ...ebooks])
       })
   
       return () => {
         socket.disconnect()
       }
     }, [ebooks])
   
     const handleSubmit = async (e) => {
       e.preventDefault()
   
       const formData = new FormData()
       formData.append('prompt', prompt)
       formData.append('author', author)
       formData.append('image', image)
   
       const res = await axios.post('http://localhost:3000/ebook', formData)
   
       socket.emit('new ebook', res.data)
     }
   
     const onDrop = (acceptedFiles) => {
       setImage(acceptedFiles[0])
     }
   
     return (
       <div>
         <h1>Criador de E-books</h1>
         <form onSubmit={handleSubmit}>
           <label htmlFor="prompt">Pergunta:</label>
           <input type="text" id="prompt" value={prompt} onChange={(e) => setPrompt(e.target.value)} />
           <label htmlFor="author">Autor:</label>
           <input type="text" id="author" value={author} onChange={(e) => setAuthor(e.target.value)} />
           <Dropzone onDrop={onDrop} multiple={false}>
             {({ getRootProps, getInputProps }) => (
               <div {...getRootProps()}>
                 <input {...getInputProps()} />
                 <p>Arraste e solte uma imagem ou clique aqui para selecionar</p>
               </div>
             )}
           </Dropzone>
           <button type="submit">Enviar</button>
         </form>
         <div>
           {ebooks.map((ebook) => (
             <div key={ebook._id}>
               <h2>{ebook.title}</h2>
               <p>{ebook.body}</p>
               <p>Por {ebook.author}</p>
               <img src={ebook.image} alt={ebook.title} />
             </div>
           ))}
         </div>
       </div>
     )
   }
   
   export default App
   ```

   

4. #### Rode o servidor Node.js:

   plaintext

   

   ```plaintext
   node server.js
   ```

   

5. #### Rode a aplicação React:

   plaintext

   

   ```plaintext
   cd ebooks
   npm start
   ```



### **Uso**

Abra o navegador e vá para `http://localhost:3000`. Você verá um formulário onde poderá digitar uma pergunta para o ChatGPT, o nome do autor do e-book e selecionar uma imagem. Clique no botão "Enviar" para enviar sua pergunta. O ChatGPT irá gerar um e-book, publicá-lo na página e enviar uma notificação em tempo real para todos os usuários conectados.





### **Servidor**

#### **server.js**

javascript



```javascript
const express = require('express')
const bodyParser = require('body-parser')
const mongoose = require('mongoose')
const fetch = require('node-fetch')
const multer = require('multer')
const socketIO = require('socket.io')

const app = express()

app.use(bodyParser.json())

mongoose.connect('mongodb://localhost:27017/ebooks', {
  useNewUrlParser: true,
  useUnifiedTopology: true
})

const EbookSchema = new mongoose.Schema({
  title: String,
  body: String,
  author: String,
  image: String
})

const Ebook = mongoose.model('Ebook', EbookSchema)

const storage = multer.diskStorage({
  destination: './uploads/',
  filename: (req, file, cb) => {
    cb(null, file.originalname)
  }
})

const upload = multer({ storage })

app.post('/ebook', upload.single('image'), async (req, res) => {
  const { prompt, author } = req.body

  const response = await fetch('https://generativelanguage.googleapis.com/v1beta2/models/chat-bison-001:generateMessage?key={{API_KEY}}', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json'
    },
    body: JSON.stringify({
      input: {
        text: prompt
      }
    })
  })

  const data = await response.json()

  const ebook = {
    title: data.candidates[0].content,
    body: data.candidates[1].content,
    author: author,
    image: req.file.originalname
  }

  const midjourneyResponse = await fetch('https://api.midjourney.com/v4/images', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Authorization': `Bearer {{API_KEY}}`
    },
    body: JSON.stringify({
      prompt: {
        text: ebook.title
      }
    })
  })

  const midjourneyData = await midjourneyResponse.json()

  ebook.image = midjourneyData.data[0].url

  const newEbook = new Ebook(ebook)

  newEbook.save()

  res.json(newEbook)
})

app.get('/ebooks', async (req, res) => {
  const ebooks = await Ebook.find()

  res.json(ebooks)
})

const io = socketIO(server)

io.on('connection', (socket) => {
  console.log('Novo usuário conectado')

  socket.on('new ebook', (ebook) => {
    io.emit('new ebook', ebook)
  })

  socket.on('disconnect', () => {
    console.log('Usuário desconectado')
  })
})

app.listen(3000, () => {
  console.log('Servidor rodando na porta 3000')
})
```



### **Cliente**

### **App.js**

javascript



```javascript
import React, { useState, useEffect } from 'react'
import axios from 'axios'
import socketIOClient from 'socket.io-client'
import Dropzone from 'react-dropzone'

const App = () => {
  const [prompt, setPrompt] = useState('')
  const [author, setAuthor] = useState('')
  const [ebooks, setEbooks] = useState([])
  const [image, setImage] = useState(null)

  useEffect(() => {
    const socket = socketIOClient('http://localhost:3000')

    socket.on('new ebook', (ebook) => {
      setEbooks([ebook, ...ebooks])
    })

    return () => {
      socket.disconnect()
    }
  }, [ebooks])

  const handleSubmit = async (e) => {
    e.preventDefault()

    const formData = new FormData()
    formData.append('prompt', prompt)
    formData.append('author', author)
    formData.append('image', image)

    const res = await axios.post('http://localhost:3000/ebook', formData)

    socket.emit('new ebook', res.data)
  }

  const onDrop = (acceptedFiles) => {
    setImage(acceptedFiles[0])
  }

  return (
    <div>
      <h1>Criador de E-books</h1>
      <form onSubmit={handleSubmit}>
        <label htmlFor="prompt">Pergunta:</label>
        <input type="text" id="prompt" value={prompt} onChange={(e) => setPrompt(e.target.value)} />
        <label htmlFor="author">Autor:</label>
        <input type="text" id="author" value={author} onChange={(e) => setAuthor(e.target.value)} />
        <Dropzone onDrop={onDrop} multiple={false}>
          {({ getRootProps, getInputProps }) => (
            <div {...getRootProps()}>
              <input {...getInputProps()} />
              <p>Arraste e solte uma imagem ou clique aqui para selecionar</p>
            </div>
          )}
        </Dropzone>
        <button type="submit">Enviar</button>
      </form>
      <div>
        {ebooks.map((ebook) => (
          <div key={ebook._id}>
            <h2>{ebook.title}</h2>
            <p>{ebook.body}</p>
            <p>Por {ebook.author}</p>
            <img src={ebook.image} alt={ebook.title} />
          </div>
        ))}
      </div>
    </div>
  )
}

export default App
```

### 



- Certifique-se de substituir as chaves `{{API_KEY}}` pelas suas chaves de API do ChatGPT e do MidJourney.

- O diretório `./uploads/` deve existir para armazenar as imagens enviadas.

- Você pode personalizar o código para atender às suas necessidades específicas.

  

### **Recursos adicionais:**

- Documentação do ChatGPT

- Documentação do MidJourney

- Documentação do Socket.IO

  

## **Conclusão**



**Conclusão**

Este é um projeto full-stack abrangente que demonstra como usar o ChatGPT e o MidJourney para criar e publicar e-books com imagens em tempo real. Este projeto é mais abrangente do que os anteriores porque usa o React Dropzone para upload de imagens e o MidJourney para gerar imagens. Você pode expandir este projeto adicionando recursos como autenticação do usuário, comentários e muito mais.

 Este projeto é adequado para iniciantes e usuários avançados que desejam construir aplicativos de geração de conteúdo poderosos.
