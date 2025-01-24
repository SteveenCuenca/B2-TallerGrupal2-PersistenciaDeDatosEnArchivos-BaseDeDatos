# B2-TallerGrupal2-PersistenciaDeDatosEnArchivos-BaseDeDatos
### Cofiguracion del usuario:
````scala
db {
  driver = "com.mysql.cj.jdbc.Driver"
  url = "jdbc:mysql://localhost:3306/estudiantes"
  user = "root"
  password = "Mariangel19."
}
````
---
### Seccion para la carga previa del archivo csv al _SQL_
```scala
object EstudiantesDAO {
  def insert(estudiantes: Estudiantes): ConnectionIO[Int] = {
    sql"""
     INSERT INTO estudiantes (nombre, edad, calificacion,genero)
     VALUES (
       ${estudiantes.nombre},
       ${estudiantes.edad},
       ${estudiantes.calificacion},
       ${estudiantes.genero}
     )
   """.update.run
  }

  def insertAll(estudiantes: List[Estudiantes]): IO[List[Int]] = {
    Database.transactor.use { xa =>
      estudiantes.traverse(t => insert(t).transact(xa))
    }
  }
}
```
---
### Estructura de la _CASE CLASS_
````scala
case class Estudiantes(
                        nombre: String,
                        edad: Int,
                        calificacion: Double,
                        genero: String
                      )

````
---
### Lectura del _CSV_ y la obtencion de resultados en sus registros
````scala
object Main extends IOApp.Simple {
  val path2DataFile2 = "src/main/resources/data/estudiantes.csv"

  val dataSource = new File(path2DataFile2)
    .readCsv[List, Estudiantes](rfc.withHeader.withCellSeparator(','))

  val estudiantes = dataSource.collect {
    case Right(estudiantes) => estudiantes
  }

  /**
   * Punto de entrada principal de la aplicación.
   * Lee temperaturas desde CSV, las inserta en la base de datos,
   * e imprime el número de registros insertados.
   *
   * @return IO[Unit] que representa la secuencia de operaciones
   */
  def run1: IO[Unit] =
    EstudiantesDAO.insertAll(estudiantes)
      .flatMap(result => IO.println(s"Registros insertados: ${result.size}"))

  // Secuencia de operaciones IO usando for-comprehension
  def run: IO[Unit] = for {
    result <- EstudiantesDAO.insertAll(estudiantes) // Inserta datos y extrae resultado con <-
    _ <- IO.println(s"Registros insertados: ${result.size}")  // Imprime cantidad
  } yield ()  // Completa la operación
}
````
#### Resultado de la ejecucion:
![image](https://github.com/user-attachments/assets/a785adce-7df3-4b81-98a1-65492f84a326)

---
## Seccion de _MySQL_
### Proceso
Primero, creamos la base de datos y la estructura de su tabla:

![image](https://github.com/user-attachments/assets/db91f431-0381-4918-be1c-d5e46dce8197)


Finalmente hacemos una consulta de los datos, luego de la inyeccion de datos co programacion, para comprobar que todo este bien.






