# Acerca de MongoDb y Mongoose:
Para utilizar MongoDb en el server debemos instalar `mongoose`: 

```javascript
>> npm install mongoose
```
Luego, la configuración en el archivo database.js es bastante sencilla: 

```javascript
import mongoose from 'mongoose';

// import dotenv from 'dotenv'
// dotenv.config();
// const {DB_NAME, DB_HOST, DB_PORT}=process.env;
// Esta es la configuración con las variables de entorno: 
// const DB_URI= `mongodb://${DB_HOST}:${DB_PORT}/${DB_NAME}`

const DB_URI= `mongodb://127.0.0.1:27017/mi-base-de-datos`; //Este es un ejemplo de la declaracion de la URI de la Db mongo

 const connectDB =  async ()=>{
  try {
   await mongoose.connect(DB_URI)
   console.log('DB conectada exitosamente ✅')
    
  } catch (error) {
    console.error(error +' algo malo pasó 🔴')
    
  }

}

export default connectDB

```
Un detalle a tener en cuenta es que no declare localhost sino la IP para conectarme, dado que por defecto intentaba conectarse por ipv6, y al declarar de la forma en que lo hice estoy utilizando ip v4.

Luego en nuestro archivo index.js:

```javascript
import app from'./src/app.js';
import conectDB from './src/database.js'


import dotenv from 'dotenv'
dotenv.config();
const {PORT}=process.env;

app.listen(PORT, ()=>{
    console.log(`Server is listening in port ${PORT} ✔️
Congratulations!! Everything is allrigt 😉!!`);
});
conectDB();
```