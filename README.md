<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="200" alt="Nest Logo" /></a>
</p>

# Teslo API

1. Clonar proyecto
2. ```npm i```
3. Clonar el archivo `.env.template` y renombrarlo a `.env`
4. Cambiar las variables de entorno
5. Levantar la base de datos
    ```
    docker-compose up -d
    ```
6. Levantar modo desarrollo: `npm run start:dev`



## Conectar Postgres con nest

Documentación: https://docs.nestjs.com/techniques/database


`npm install --desarrollo decoradores   paquete_TypeOrm  driver`

`npm install --save  @nestjs/typeorm    typeorm          mysql2`

TypeOrm es agnóstico a la base de datos 

Un **ORM** es una pieza de software que nos permite interactuar con nuestra BD sin la necesidad de conocer lenguaje SQL, que es lenguaje de consultas, todo esto utilizando el paradigma POO.
Los **ORM** se encargan de traducir nuestras instrucciones en el lenguaje de programación que estemos utilizando a una sentencia SQL que el gestor de BD pueda comprender y ejecutar.
Existe un ORM para cada lenguaje, por ejemplo para java está Hibernate y para el caso de Javascript existe sequelize, prisma, TypeOrm, entre otros.

1. instalar typeOrm y sus decoradores `npm i --save @nestjs/typeorm typeorm`
2. Instalar el driver, en el caso de postgress `npm install pg --save` o como se muestra al comienzo en una sola línea `npm install --save @nestjs/typeorm typeorm pg`,

3. Configurar variables de entorno en el archivo `.env` 
    ```
    DB_PASSWORD=MySecretPassWord@as2
    DB_NAME=TesloDB
    DB_HOST=localhost
    DB_PORT=5432
    DB_USERNAME=postgres
    ```
4. En el app.module agregar `TypeOrmModule.forRoot({variables})`

    ```
    @Module({
      imports: [
        ConfigModule.forRoot(),

        TypeOrmModule.forRoot({
          type: 'postgres',
          host: process.env.DB_HOST,
          port: +process.env.DB_PORT,
          database: process.env.DB_NAME,
          username: process.env.DB_USERNAME,
          password: process.env.DB_PASSWORD,
          autoLoadEntities: true,   //carga automaticamente las entidades
          synchronize: true //sincroniza todos los cambios automaticamente
        })
      ],
      controllers: [],
      providers: [],
    })
    export class AppModule {}
    ```

## queryBuilder

Permite construir nuestras propias querys.
```
    const queryBuilder = this.productRepository.createQueryBuilder()

    product = await queryBuilder
    .where('UPPER(title) =:title or slug =:slug', {
      title: term.toUpperCase(),
      slug: term.toLowerCase(),
    }).getOne();
```