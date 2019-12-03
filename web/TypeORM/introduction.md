# Introduction

 TypeORM can reate database tables for you and find / insert / update / delete your data without the pain of having to write lots of hardly maintainable SQL queries.

## Create a model

TypeORM create a database table through the models.

```ts
export class Photo {
    id: number;
    name: string;
    description: string;
    filename: string;
    views: number;
    isPublished: boolean;
}
```

Database tables are created from models which you define as entities.

## Create an entity

Entity is your model decorated by an @Entity decorator.
You can load/insert/update/remove and perform other operations with Entities.

```ts
import {Entity} from "typeorm";

@Entity()
export class Photo {
    id: number;
    name: string;
    description: string;
    filename: string;
    views: number;
    isPublished: boolean;
}
```

## Adding table columns

Decorate an entity's properties you want to make into a column with a @Column decorator.

```ts
import {Entity, Column} from "typeorm";

@Entity()
export class Photo {

    @Column()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### Creating a primary column

Each entity must have at least one primary key column.
We can add a primary key to `id` field.

```ts
import {Entity, Column} from "typeorm";

@Entity()
export class Photo {

    @PrimaryColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### Creating an auto generated column

```ts
import {Entity, Column} from "typeorm";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @Column()
    description: string;

    @Column()
    filename: string;

    @Column()
    views: number;

    @Column()
    isPublished: boolean;
}
```

### Column data types


```ts
import {Entity, Column} from "typeorm";

@Entity()
export class Photo {

    @PrimaryGeneratedColumn()
    id: number;

    @Column({
      length: 100
    })
    name: string;

    @Column("text")
    description: string;

    @Column()
    filename: string;

    @Column("double")
    views: number;

    @Column()
    isPublished: boolean;
}
```

## Creating a connection to the database

```ts

// index.ts

import "reflect-metadata";
import {createConnection} from "typeorm";
import {Photo} from "./entity/Photo";

createConnection({
    type: "mysql",
    host: "localhost",
    port: 3306,
    username: "root",
    password: "admin",
    database: "test",
    entities: [
        Photo
        // or simply use directory which stores entities, like
        __dirname + "/entity/*.js"
    ],
    synchronize: true,
    logging: false
}).then(connection => {
    // here you can start to work with your entities
}).catch(error => console.log(error));
```

## Creating and inserting a photo into the database

```ts
import {createConnection} from "typeorm";
import {Photo} from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;

    await connection.manager.save(photo);
    console.log("Photo has been saved");

}).catch(error => console.log(error));
```

## Using Repositories

Each entity has its own repository which handles all operations with its entity.

```ts
import {createConnection} from "typeorm";
import {Photo} from "./entity/Photo";

createConnection(/*...*/).then(async connection => {

    let photo = new Photo();
    photo.name = "Me and Bears";
    photo.description = "I am near polar bears";
    photo.filename = "photo-with-bears.jpg";
    photo.views = 1;
    photo.isPublished = true;

    let photoRepository = connection.getRepository(Photo);

    await photoRepository.save(photo);
    console.log("Photo has been saved");

    let savedPhotos = await photoRepository.find();
    console.log("All photos from the db: ", savedPhotos);

}).catch(error => console.log(error));
```

## Creating a ont-to-one relation between two entities

Create a new class in PhotoMetadata.ts and create the relationship between `Photo` and `PhotoMetadata`.
Using @JoinColumn decorator is required on the owner side of the relationship.
In this example, the owner of the relation is PhotoMetadata, and Photo doesn't know anything about PhotoMetadata.
The owning side of a relationship contains a column with a foreign key in the database. 

```ts
import {Entity, Column, PrimaryGeneratedColumn, OneToOne, JoinColumn} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class PhotoMetadata {

    @PrimaryGeneratedColumn()
    id: number;

    @Column("int")
    height: number;

    @Column("int")
    width: number;

    @Column()
    orientation: string;

    @Column()
    compressed: boolean;

    @Column()
    comment: string;

    @OneToOne(type => Photo)
    @JoinColumn()
    photo: Photo;
}
```

The table will look like this:

```ts
+-------------+--------------+----------------------------+
|                     photo_metadata                      |
+-------------+--------------+----------------------------+
| id          | int(11)      | PRIMARY KEY AUTO_INCREMENT |
| height      | int(11)      |                            |
| width       | int(11)      |                            |
| comment     | varchar(255) |                            |
| compressed  | boolean      |                            |
| orientation | varchar(255) |                            |
| photoId     | int(11)      | FOREIGN KEY                |
+-------------+--------------+----------------------------+
```


When you are going to save the relationship:

```ts
    // first we should save a photo
    await photoRepository.save(photo);

    // photo is saved. Now we need to save a photo metadata
    await metadataRepository.save(metadata);
```

Now, `Photo` doesn't know anything about `PhotoMetadata`.
If we want to add the inverse side of relationship, say, let `Photo` knows `PhotoMetadata`, what shoud we do ?

```ts
import {Entity, Column, PrimaryGeneratedColumn, OneToOne, JoinColumn} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class PhotoMetadata {

    /* ... other columns */

    @OneToOne(type => Photo, photo => photo.metadata)
    @JoinColumn()
    photo: Photo;
}
```

```ts
import {Entity, Column, PrimaryGeneratedColumn, OneToOne} from "typeorm";
import {PhotoMetadata} from "./PhotoMetadata";

@Entity()
export class Photo {

    /* ... other columns */

    @OneToOne(type => PhotoMetadata, photoMetadata => photoMetadata.photo)
    metadata: PhotoMetadata;
}
```

## Loading objects with their relations

```ts
import {createConnection} from "typeorm";
import {Photo} from "./entity/Photo";
import {PhotoMetadata} from "./entity/PhotoMetadata";

createConnection(/*...*/).then(async connection => {

    /*...*/
    let photoRepository = connection.getRepository(Photo);
    let photos = await photoRepository.find({ relations: ["metadata"] });

}).catch(error => console.log(error));
```

There is another API `QueryBuilder` to build a more complex query.

## Using cascades to automatically save related objects

Using cascade allows us not to separately save photo and separately save metadata objects now.
Now we can simply save a photo object, and the metadata object will be saved automatically because of cascade options.

```ts
export class Photo {
    /// ... other columns

    @OneToOne(type => PhotoMetadata, metadata => metadata.photo, {
        cascade: true,
    })
    metadata: PhotoMetadata;
}
```

## Creating a many-to-one / one-to-many relation

Let's say a photo has one author, and each author can have many photos. First, let's create an Author class:

```ts
import {Entity, Column, PrimaryGeneratedColumn, OneToMany, JoinColumn} from "typeorm";
import {Photo} from "./Photo";

@Entity()
export class Author {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @OneToMany(type => Photo, photo => photo.author) // note: we will create author property in the Photo class below
    photos: Photo[];
}
```

And there must be `ManyToOne` on the other side:

```ts
import {Entity, Column, PrimaryGeneratedColumn, ManyToOne} from "typeorm";
import {PhotoMetadata} from "./PhotoMetadata";
import {Author} from "./Author";

@Entity()
export class Photo {

    /* ... other columns */

    @ManyToOne(type => Author, author => author.photos)
    author: Author;
}
```
In many-to-one / one-to-many relation, the owner side is always many-to-one. 
It means that the class that uses @ManyToOne will store the id of the related object (authorId).

## Creating a many-to-many relation

Let's say a photo can be in many albums, and each album can contain many photos. Let's create an Album class:

```ts
import {Entity, PrimaryGeneratedColumn, Column, ManyToMany, JoinTable} from "typeorm";

@Entity()
export class Album {

    @PrimaryGeneratedColumn()
    id: number;

    @Column()
    name: string;

    @ManyToMany(type => Photo, photo => photo.albums)
    @JoinTable()
    photos: Photo[];
}
```

@JoinTable is required to specify that this is the owner side (own the ids) of the relationship.

And the other sides:

```ts
export class Photo {
    /// ... other columns

    @ManyToMany(type => Album, album => album.photos)
    albums: Album[];
}
```

After you run the application, the ORM will create a `album_photos_photo_albums` junction table:

```ts
+-------------+--------------+----------------------------+
|                album_photos_photo_albums                |
+-------------+--------------+----------------------------+
| album_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
| photo_id    | int(11)      | PRIMARY KEY FOREIGN KEY    |
+-------------+--------------+----------------------------+
```

## Using QueryBuilder

For example:

```ts
let photos = await connection
    .getRepository(Photo)
    .createQueryBuilder("photo") // first argument is an alias. Alias is what you are selecting - photos. You must specify it.
    .innerJoinAndSelect("photo.metadata", "metadata")
    .leftJoinAndSelect("photo.albums", "album")
    .where("photo.isPublished = true")
    .andWhere("(photo.name = :photoName OR photo.name = :bearName)")
    .orderBy("photo.id", "DESC")
    .skip(5)
    .take(10)
    .setParameters({ photoName: "My", bearName: "Mishka" })
    .getMany();
```
