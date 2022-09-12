<p align="center">
  <a href="http://nestjs.com/" target="blank"><img src="https://nestjs.com/img/logo-small.svg" width="120" alt="Nest Logo" /></a>
</p>

## Creating a project Step by Step

1. Crear el proyecto `nest new project-name`.
    * Borrar los archivos innecesarios (dejar sólo el module)


2. Configurar variables de entorno:
    * Crear archivo .env con las variables de entorno.
    * Instalar `npm i @nestjs/config`
    * Configurar en el módulo 
        ```
        import { Module } from '@nestjs/common';
        import { ConfigModule } from '@nestjs/config';

        @Module({
        imports: [ConfigModule.forRoot()],
        })
        export class AppModule {}
        ```

    * Clonar `.env`, renombrarlo a `.env.template`.

    **`NOTA:` Siempre que se haga un cambio en las variables de entorno hay que bajar y volver a subir la app para que sean reconocidos los cambios.**

3. Configurar la base de datos. En caso de manejarla con docker, se debe:
    * crear archivo `docker-compose.yaml`
    * Levantar la base de datos `docker-compose up -d`. 
    
        Debe empezar a correr en docker.
    * Instalar `npm install --save @nestjs/typeorm typeorm pg`.
    Esto realiza la instalación de typeOrm, sus decoradores y el driver de la bd que se va a utilizar, en este caso postgres, su driver es **pg**. 
    * En el app.module agregar `TypeOrmModule.forRoot({variables})` 
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

4. Ignorar archivos en el `.gitignore`

5. Crear un nuevo recurso, un nuevo crud `nest g res products --no-spec`

6. Definir las entity. 
`Entity` es la representación de nuestro objeto en la base de datos, una tabla.

7. En products.module importar `TypeOrmModule.forFeature([entity])`, dentro del forFeature voy a definir un arreglo con todas las entidades que este módulo está definiendo.

    ```
    @Module({
    controllers: [ProductsController],
    providers: [ProductsService],
    imports: [
        TypeOrmModule.forFeature([Product])
    ]
    })
    export class ProductsModule {}
    ```

8. Instalar class-validator y class-transformer, para realizar los decoradores para realizar validaciones.

    ``` 
    npm i class-validator class-transformer
    ```
9. En el main poner la configuración del GlobalPipes

    ```
    import { ValidationPipe } from '@nestjs/common';

    app.useGlobalPipes( 
        new ValidationPipe({
            whitelist: true,
            forbidNonWhitelisted: true,
        })
    );
    ```

10. Agregegar los atributos y sus validaciones a `create-product.dto.ts`

11. Crear los servicios, create, update, get y delete.
Esto lo hacemos con el patron repositorio, en el cual creamos el productRepository que será del tipo Repository, este contendrá las acciones, save, create, delete....
luego se crear, eliminar, etc, se debe guardar y retornar el elemento.

    ```
    constructor(
        @InjectRepository(Product)
        private readonly productRepository: Repository<Product>
    ){}

    async create(createProductDto: CreateProductDto) {
        
        try {

        const product = this.productRepository.create(createProductDto)
        await this.productRepository.save(product)
        return product;

        } catch (error) {
        console.log(error);
        this.handleDBExceptions(error)
        
        }
    }
    ```


12. Se crea una función para manejar los errores y así poder utilizarla en el post, put, delete y en todos los servicios que se vayan a crear.

    En el service. El logger nos permite ver los errores de una manera más ordenada, de la manera en que nest muestra sus logs.


    ```
    private readonly logger = new Logger('ProductsService')
    ```

    Y podra ser usada de la siguiente manera: 

    ```
    private handleDBExceptions(error: any){

        if(error.code === '23505'){
        throw new BadRequestException(error.detail)
        }
        this.logger.error(error)
        throw new InternalServerErrorException('Unexpected error, check logs')
        
    }
    ```

***`NOTA:` Para poder utilizar el replace all es necesario ir al tsconfig.json y cambiar la versión del target.***

13. Para crear el slug a partit del title y/o validarlo para que quede de la forma deseada se crea un método en la entidad que será ejecutado anter de realizar la inserción, para esto se utiliza el `@BeforeInsert()` , de igual manera se puede utilizar el `@BeforeUpdate`
    ```
        @BeforeInsert()  //antes de insertar se ejecuta ese método.
        checkSlugInsert(){
            
            if(!this.slug){
                this.slug = this.title;
            }
            this.slug = this.slug
            .toLocaleLowerCase()
            .replaceAll(' ', '_')
            .replaceAll("'",'')
        }
    ```

14. A partir del `productRepository` se crean los servicios, create, read, update y delete.
**Recordar el async y el await y el manejo de errores**

15. En el controler recordar parsear el parámetro para garantizar que llegue de la forma esperada `(@Param('id', ParseUUIDPipe)`

16. para crear la paginación, lo creamos en un módulo común, para esto generamos un nuevo módulo con `nest g mo common`

17. Los Query Parámeters vienen como string, para realizar la conversión se puede usar de la forma hecha en la app de pokemon que era en el main, en el app.useGlobalPipes.

    ```
    transform: true,
    transformOptions: {
    enableImplicitConversion: true
    ```
Esto transforma la clase al tipo de dato que le especifico, ejemplo **limit?: number;** es un Query parameter que llegará como string, pero al decirle que es de tipo number el transformOptions lo transformará al tipo de dato indicado.

La segunda forma es con el objeto Number propio de javascript, y este hará la transformación.

```
@Type(() => Number)
```

* instalar `npm i uuid` y `npm i -D @types/uuid` tiene funciones para evaluarlo.

    importarmos en el service **import { validate as isUUID } from "uuid";** , de este modo se puede usar isUUID(term) para validar si es un UUID.
    **`NOTA:` También existe isUUID en class-validator** 

* con queryBuilder se puede realizar consultas por slug o por título.

    Permite construir nuestras propias querys.
    ```
        const queryBuilder = this.productRepository.createQueryBuilder()

        product = await queryBuilder
        .where('UPPER(title) =:title or slug =:slug', {
        title: term.toUpperCase(),
        slug: term.toLowerCase(),
        }).getOne();
    ```


