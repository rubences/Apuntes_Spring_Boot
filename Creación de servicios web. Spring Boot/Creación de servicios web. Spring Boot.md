# Creación de servicios web. Spring Boot
¿Qué son los servicios web?
Un servicio web es una aplicación que se encuentra en el lado servidor y permite que otra aplicación cliente conecte con ella a través de Internet para el intercambio de información utilizando el protocolo HTTP.

Una de las principales características de los servicios web es que no es necesario que ambas aplicaciones (servidor y cliente) estén escritas en el mismo, lo que hace que la interoperabilidad sea máxima. Por ejemplo, podríamos crear un servicio web en Python y utilizarlo conectándonos desde una aplicación móvil con Android, desde otra aplicación programada en Java o incluso desde otro servicio web escrito con .NET.

Además, utilizan el protocolo HTTP para el intercambio de información, lo que significa que la conexión se establece por el puerto 80 que es el mismo que utilizan los navegadores y que es prácticamente seguro que se encuentre abierto en cualquier organización protegida por firewall. Esto hace que no sea necesario tener especial cuidado abriendo puertos innecesarios para poder conectarnos a ellos. Antes de la llegada de los servicios web como los conocemos ahora existían otros protocolos más complicados que requerían de servicios y puertos adicionales, incrementando el riesgo de ataques en las organizaciones que los ponían en marcha.

web_service_internet.jpg
Figure 1: Esquema de funcionamiento de un servicio web
REST Web Service
Los Servicios Web REST son Servicios Web que cumplen una serie de requisitos según un patrón de arquitectura definida hacia el año 2000 y que se ha extendido siendo el patrón predominante a la hora de implementar este tipo de aplicaciones.

Básicamente consiste en seguir una serie de reglas que definen dicha arquitectura. Entre ellas están el uso del procotolo HTTP por ser el más extendido a lo largo de Internet en la actualidad. Además, cada recurso del servicio web tiene que ser identificado por una dirección web (una URL) siguiendo una estructura determinada. Además, la respuesta tendrá que tener una estructura determinada en forma de texto que normalmente vendrá en alguno de los formatos abiertos más conocidos como XML o JSON.


Figure 2: Ejemplos de URLs que definen el acceso a las operaciones de un servicio web REST
Esas URLs y su estructura son lo que definen lo que se conoce como la API del Servicio Web, que son las diferentes operaciones a las que los clientes tienen acceso para comunicarse con el mismo. En este caso se trata de una API Web.

Web API
Una Web API es una API (Application Programming Interface) implementada para un Servicio Web de forma que éste puede ser accesible mediante el protocolo HTTP, en principio por cualquier cliente web (navegador) aunque existen librerías que permiten que cualquier tipo de aplicación (escritorio, web, móvil, otros servicios web, . . .) accedan a la misma para comunicarse con dicho servicio web.

La Web API es una de las partes de los Servicios Web que, tal y como comentabamos anteriormente, mejoran sustancialmente la interoperabilidad de éstos con los potenciales clientes ya que permiten que sólo haya que implementar un único punto de entrada para comunicarse con el servicio web independientemente del tipo de aplicación que lo haga. De esa manera el desarrollador del Servicio Web define la lógica de negocio en el lado servidor y los diferentes clientes que quieran comunicarse con el mismo lo hacen a través de la Web API realizando solicitudes a las diferentes URLs que definen las operaciones disponibles.


Figure 3: Arquitectura de aplicación web sin API

Figure 4: Arquitectura de aplicación web con API
JSON
{
  "firstName": "John",
  "lastName": "Smith",
  "isAlive": true,
  "age": 25,
  "address": {
    "streetAddress": "21 2nd Street",
    "city": "New York",
    "state": "NY",
    "postalCode": "10021-3100"
  },
  "phoneNumbers": [
    {
      "type": "home",
      "number": "212 555-1234"
    },
    {
      "type": "office",
      "number": "646 555-4567"
    },
    {
      "type": "mobile",
      "number": "123 456-7890"
    }
  ],
  "children": [],
  "spouse": null
}
Desarrollo de servicios web con Spring Boot
En el punto anterior sobre Creación de aplicaciones web. Spring Boot vimos cómo comenzar el desarrollo de una aplicación web interactiva. Ahora se trata de implementar un proyecto muy similar, puesto que se puede considerar una aplicación web pero en este caso implementaremos un Controlador REST en lugar de un controlador web.

Podemos seguir el mismo guión que en el punto anterior hasta el momento en que se define el @Controller y se empieza a trabajar con las plantillas HTML. Ahora se trata de implementar servicios web por lo que definiremos, en su lugar, una clase que hará de @RestController y no habrá plantillas HTML puesto que la comunicación se hará utilizando JSON (conversión que Spring Boot hará automáticamente) y no será usuario-máquina sino máquina-maquina.

Partimos entonces de un proyecto de aplicación con Spring Initializr:

Configuración del servidor
Lo primero de todo será editar el fichero de configuración del proyecto para personalizarlo a nuestro caso. En el siguiente ejemplo estaríamos configurando la aplicación para conectar con una base de datos MySQL:

application.properties
# Configuracion para el acceso a la Base de Datos
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
 
# Puerto donde escucha el servidor una vez se inicie
server.port=8080
 
# Datos de conexion con la base de datos MySQL
spring.datasource.url=jdbc:mysql://localhost:3306/myshoponline
spring.datasource.username=myshopuser
spring.datasource.password=mypassword
spring.datasource.driverClassName=com.mysql.jdbc.Driver
Pero también nos podría interesar utilizar una base de datos H2:

application.properties
# Configuracion para el acceso a la Base de Datos
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.globally_quoted_identifiers=true
 
# Puerto donde escucha el servidor una vez se inicie
server.port=8080
 
# Datos de conexion con la base de datos H2
spring.datasource.url=jdbc:h2:file:/Ruta/al/fichero/myshoponline.db
spring.datasource.driverClassName=org.h2.Driver
spring.datasource.username=sa
spring.datasource.password=password
spring.jpa.database-platform=org.hibernate.dialect.H2Dialect
spring.h2.console.enabled=true
Hay que tener en cuenta que la propiedad spring.jpa.hibernate.ddl-auto se utiliza para que la base de datos se genere automáticamente en cada arranque de la aplicación. Esto nos interesará cuando estemos en desarrollo pero no cuando queramos desplegarla en producción. Por lo que tendremos que tener cuidado y controlar el valor de dicha propiedad.

none: Para indicar que no queremos que genere la base de datos
update: Si queremos que la genere de nuevo en cada arranque
create: Si queremos que la cree pero que no la genere de nuevo si ya existe
Definir la base de datos
Hay que tener en cuenta que Spring utiliza por debajo el framework de Hibernate para trabajar con la Base de Datos. Eso nos va a permitir trabajar con nuestras clases Java directamente sobre la Base de Datos, ya que será Hibernate quién realizará el mapeo entre el objeto Java (y sus atributos) y la tabla de MySQL (y sus columnas) a la hora de realizar consultas, inserciones, modificaciones o borrados. E incluso a la hora de crear las tablas, puesto que bastará con definir nuestro modelo de clases con las anotaciones apropiadas para que Spring pueda crearlas en base a éstas (y porque tenemos la opción spring.jpa.hibernate.ddl-auto=update en el fichero de configuración). Cuando ya no queramos que Spring genere automáticamente la base de datos en cada arranque (por ejemplo, en producción), tendremos que poner cambiar esa opción a valor none.

Hay que tener en cuenta que, si hemos configurado el proyecto para hacer uso de una base de datos H2, no será necesario realizar este paso, puesto que la base de datos se creará automáticamente la primera vez que se inicie la aplicación. Podemos omitir este apartado.
Simplemente tendremos que crear la base de datos. Y ya de paso aprovecharemos para crear un usuario con el que la aplicación web se conectará (de esa manera evitamos tener que configurar el acceso usando el usuario root).

CREATE DATABASE myshoponline;
CREATE USER myshopuser IDENTIFIED BY 'mypassword';
GRANT ALL PRIVILEGES ON myshoponline.* TO myshopuser;
Usaremos myshopuser como usuario y mypassword como usuario y contraseña en el fichero de configuración (application.properties) del proyecto.

Definir el modelo de datos
Como Spring Boot utiliza Hibernate como libreria ORM (Object-Relationship Mapping), para definir el modelo de datos de nuestra base de datos, nos bastará con escribir las clases Java que representarán a los datos en nuestra aplicación web. A través de las anotaciones que veremos a continuación, le daremos las instrucciones a Spring acerca de cómo crear la base de datos de forma transparente para nosotros.

Así, simplemente tenemos que crear la clase con los atributos y métodos que queramos y añadir las anotaciones que orientarán a Hibernate para saber a qué tabla corresponden los objetos de la clase y a qué columnas sus atributos.

Usaremos, además, la librería lombok para la generación automática de getters, setters y constructores.

import lombok.*;
 
import javax.persistence.*;
import java.time.LocalDateTime;
/**
 * Producto de la tienda online
 *
 * @author Santiago Faci
 * @version curso 2021
 */
@Data
@AllArgsConstructor
@NoArgsConstructor
@Entity(name = "products")
public class Product {
 
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    @Column
    private String name;
    @Column
    private String description;
    @Column
    private String category;
    @Column
    private float price;
    @Column(name = "creation_date")
    private LocalDateTime creationDate;
}
Recordad que todas las anotaciones Java en el ejemplo anterior son clases que pertenecen al paquete 'javax.persistence'. Tened cuidado de no importar las mismas clases que existen en otros paquetes, aunque estén relacionados con Spring
El acceso a la base de datos
Ahora creamos la interface donde se definirán los métodos que permitirán acceder a la Base de Datos. En este caso nos basta con definir las cabeceras de los mismos, puesto que se trata de una interface. Será el framework el que se encargue de su implementación. En este caso hemos definido métodos para obtener todas las puntuaciones y otro para obtener las que tengan una puntuación determinada. Además, podremos contar con que tenemos las operaciones que nos permiten registrar/modificar (save) y eliminar (delete) información de la Base de Datos.

/** 
 * Repositorio de Productos
 * @author Santiago Faci
 * @version curso 2021
 */
@Repository
public interface ProductRepository extends CrudRepository<Product, Long> {
 
    Set<Product> findAll();
    Set<Product> findByCategory(String category);
}
Implementación de la lógica de negocio: Los Services
Los Services serán la capa de nuestra aplicación web donde implementaremos toda la lógica de negocio.

Definiremos una interface con todos los métodos que necesitemos:

public interface ProductService {
 
    Set<Product> findAll();
    Set<Product> findByCategory(String category);
    Optional<Product> findById(long id);
    Product addProduct(Product product);
    Product modifyProduct(long id, Product newProduct);
    void deleteProduct(long id);
}
Que implementaremos en la clase ProductServiceImpl

@Service
public class ProductServiceImpl implements ProductService {
 
    @Autowired
    private ProductRepository productRepository;
 
    @Override
    public Set<Product> findAll() {
        return productRepository.findAll();
    }
 
    @Override
    public Set<Product> findByCategory(String category) {
        return productRepository.findByCategory(category);
    }
 
    @Override
    public Optional<Product> findById(long id) {
        return productRepository.findById(id);
    }
 
    @Override
    public Product addProduct(Product product) {
        return productRepository.save(product);
    }
 
    @Override
    public Product modifyProduct(long id, Product newProduct) {
        Product product = productRepository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
        newProduct.setId(product.getId());
        return productRepository.save(newProduct);
    }
 
    @Override
    public void deleteProduct(long id) {
        productRepository.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
        productRepository.deleteById(id);
    }
}
Implementación del controller
Antes de continuar, es muy conveniente leerse el siguiente artículo sobre los diferentes métodos HTTP en servicios web REST. Explica muy claramente cómo deben ser las diferentes operaciones que se pueden llevar a cabo sobre los recursos del sistema.

Y a continuacuón, en esta ocasión definiremos un controlador REST. En él se han definido diferentes endpoints que permite ejecutar las siguientes operaciones:

Obtener todos los productos: Método GET que devuelve toda la colección
Obtener todos los productos de una categoría determinada: Método GET que devuelve la colección filtrada por el campo categoria
Obtener un producto determinado: Método GET que devuelve un objeto determinado utilizando un parámetro Path
Registrar un nuevo producto: Método POST que registra un nuevo producto en la base de datos
Modificar un producto: Método PUT que modifica un producto
Eliminar un producto: Método DELELET que elimina un producto existente
Como veremos, algunas de las operaciones devuelven un error controlado (mediante un gestor de excepciones que se ha definido al final del controlador) cuando el producto solicitado no existe. En esos casos, se devuelve además una respuesta definida en la clase Response para notificar el código de error y mensaje de negocio, a parte del código de estado HTTP correspondiente.

Para entender el siguiente fragmento de código conviene tener en cuenta lo siguiente:

Cada método anotado define un endpoint que podrá ser invocado por otra aplicación
Las anotaciones @GetMapping, @PostMapping, @PutMapping, @PatchMapping, @DeleteMapping definen el método (GET, POST, PUT, PATCH, DELETE) y la URL de dicho endpoint.
Si el endpoint debe utilizar Path Params vendrán definidos en la URL y también en el método como @PathVariable
Si el endpoint debe utilizar Query Params vendrán definidos solamente en el método como @RequestParam
Si el endpoint debe utilizar Body Params vendrán definidos solamente en el método como @RequestBody
La respuesta será siempre un objeto ResponseEntity que contedrá la información de respuesta y un código de estado HTTP asociado (más información)
@RestController
public class ProductController {
 
    @Autowired
    private ProductService productService;
 
    @GetMapping("/products")
    public ResponseEntity<Set<Product>> getProducts(@RequestParam(value = "category", defaultValue = "") String category) {
        Set<Product> products = null;
        if (category.equals(""))
            products = productService.findAll();
        else
            products = productService.findByCategory(category);
 
        return new ResponseEntity<>(products, HttpStatus.OK);
    }
 
    @GetMapping("/products/{id}")
    public ResponseEntity<Product> getProduct(@PathVariable long id) {
        Product product = productService.findById(id)
                .orElseThrow(() -> new ProductNotFoundException(id));
 
        return new ResponseEntity<>(product, HttpStatus.OK);
    }
 
    @PostMapping("/products")
    public ResponseEntity<Product> addProduct(@RequestBody Product product) {
        Product addedProduct = productService.addProduct(product);
        return new ResponseEntity<>(addedProduct, HttpStatus.OK);
    }
 
    @PutMapping("/products/{id}")
    public ResponseEntity<Product> modifyProduct(@PathVariable long id, @RequestBody Product newProduct) {
        Product product = productService.modifyProduct(id, newProduct);
        return new ResponseEntity<>(product, HttpStatus.OK);
    }
 
    @DeleteMapping("/products/{id}")
    public ResponseEntity<Response> deleteProduct(@PathVariable long id) {
        productService.deleteProduct(id);
        return new ResponseEntity<>(Response.noErrorResponse(), HttpStatus.OK);
    }
 
    @ExceptionHandler(ProductNotFoundException.class)
    @ResponseBody
    @ResponseStatus(HttpStatus.NOT_FOUND)
    public ResponseEntity<Response> handleException(ProductNotFoundException pnfe) {
        Response response = Response.errorResonse(NOT_FOUND, pnfe.getMessage());
        return new ResponseEntity<>(response, HttpStatus.NOT_FOUND);
    }
}
Como se puede ver, al final del controlador, se ha definido un método que sirve de ejemplo para ver cómo tratar las excepciones que se puedan producir. En este caso será necesario implementar la clase ProductNotFoundException que define la excepción para los casos en los que no se encuentra el objeto requerido.

public class ProductNotFoundException extends RuntimeException {
 
    public ProductNotFoundException() {
        super();
    }
 
    public ProductNotFoundException(String message) {
        super(message);
    }
 
    public ProductNotFoundException(long id) {
        super("Product not found: " + id);
    }
}
También necesitaremos implementar la clase Response que usamos como respuesta genérica cuando lo que hay que responder no es información sino que es la confirmación de que una operación se ha ejecutado correctamente o bien un error porque algo no ha ocurrido como se esperaba.

@Data
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class Response {
 
    public static final int NO_ERROR = 0;
    public static final int NOT_FOUND = 101;
 
    public static final String NO_MESSAGE = "";
 
    private Error error;
 
    @Data
    @AllArgsConstructor(access = AccessLevel.PRIVATE)
    static class Error {
        private long errorCode;
        private String message;
    }
 
    public static Response noErrorResponse() {
        return new Response(new Error(NO_ERROR, NO_MESSAGE));
    }
 
    public static Response errorResonse(int errorCode, String errorMessage) {
        return new Response(new Error(errorCode, errorMessage));
    }
}
Trazabilidad. Logs de aplicación
Si queremos mantener la trazabilidad de la ejecución de nuestra aplicación (y esto sería válido tanto para la aplicación web como para el proyecto de servicio web que estamos haciendo ahora), tenemos que configurar cómo queremos que se registren los sucesos y trazas de la ejecución.

Por defecto, cuando ejecutamos la aplicación en modo desarrollo, y también ocurre asi cuando se hace en producción, Spring Boot lanza por pantalla las trazas de ejecución con una configuración predeterminada. Pero tenemos la opción de configurar como queremos que sean esas trazas y si queremos que también se genere un log físico en disco, usando la librería logback (que es la sucesora de la ya conocida librería log4j).

Para eso, simplemente tenemos que crear un fichero llamado logback-spring.xml en la carpeta resources del proyecto. Y a continuación se muestra un ejemplo de cómo tendría que quedar ese fichero para tener una traza por consola al ejecutar y el comando y, al mismo tiempo, que esa traza quedara almacenada en un fichero en disco de forma que éste rotara cada día o cada vez que llegara a un tamaño determinado (fijado en 10 MB).

<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <!-- Propiedades que se usará para indicar dónde almacenar los logs y cómo se llama el fichero -->
    <property name="LOG_DIR" value="logs" />
    <property name="LOG_NAME" value="myshop" />
 
    <!-- Configuración del log que aparece por consola: Console appender -->
    <appender name="Console"
              class="ch.qos.logback.core.ConsoleAppender">
        <layout class="ch.qos.logback.classic.PatternLayout">
            <!-- Configuración de la traza -->
            <Pattern>
                %white(%d{ISO8601}) %highlight(%-5level) [%blue(%t)] %-60.60yellow(%C{20}): %msg%n%throwable
            </Pattern>
        </layout>
    </appender>
 
    <!-- Configuración para que se almacene el log en un fichero: File Appender -->
    <appender name="RollingFile"
              class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${LOG_DIR}/${LOG_NAME}.log</file>
        <encoder
                class="ch.qos.logback.classic.encoder.PatternLayoutEncoder">
            <Pattern>%d %p %C{1.} [%t] %m%n</Pattern>
        </encoder>
 
        <!-- Política de rotado de logs: diario y cuando el fichero llegue a los 10 MB -->
        <rollingPolicy
                class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>${LOG_DIR}/${LOG_NAME}-%d{yyyy-MM-dd}.%i.log</fileNamePattern>
            <timeBasedFileNamingAndTriggeringPolicy
                    class="ch.qos.logback.core.rolling.SizeAndTimeBasedFNATP">
                <maxFileSize>10MB</maxFileSize>
            </timeBasedFileNamingAndTriggeringPolicy>
        </rollingPolicy>
    </appender>
 
    <!-- Define el nivel de log para cada appender -->
    <root level="info">
        <appender-ref ref="RollingFile" />
        <appender-ref ref="Console" />
    </root>
</configuration>
Asi es como quedaría el fichero log resultante con las trazas de ejecución de la aplicación:

santi@zenbook:$ cd logs 
santi@zenbook:$ ls -la
total 56
drwxr-xr-x   3 santi  staff    96B Mar  2 21:36 .
drwxr-xr-x  14 santi  staff   448B Mar  2 21:36 ..
-rw-r--r--   1 santi  staff    28K Mar  2 21:51 myshop.log
Hasta el momento, la mayoría de las trazas que se registran las emite el propio framework Spring Boot. Pero nosotros tenemos la oportunidad de registrar las que consideremos oportunas utilizando la clase LoggerFactory que permite obtener una instancia de un objecto Logger para registrar trazas y dejar asi constancia de cualquier evento de importancia.

private final Logger logger = LoggerFactory.getLogger(ProductController.class);
Por ejemplo, a continuación se registran un par de trazas para que quede constancia de que se ha invocado a la operación que permite listar los productos del catálogo:

@GetMapping("/products")
    public ResponseEntity<Set<Product>> getProducts(@RequestParam(value = "category", defaultValue = "") String category) {
        logger.info("inicio getProducts");
        Set<Product> products = null;
        if (category.equals(""))
            products = productService.findAll();
        else
            products = productService.findByCategory(category);
 
        logger.info("fin getProducts");
        return new ResponseEntity<>(products, HttpStatus.OK);
    }
También en el caso de que se produzca alguna excepción, será interesante registrar una traza e incluso podremos incluir la propia excepción:

@ExceptionHandler(ProductNotFoundException.class)
@ResponseBody
@ResponseStatus(HttpStatus.NOT_FOUND)
public ResponseEntity<Response> handleException(ProductNotFoundException pnfe) {
    Response response = Response.errorResonse(NOT_FOUND, pnfe.getMessage());
    logger.error(pnfe.getMessage(), pnfe);
    return new ResponseEntity<>(response, HttpStatus.NOT_FOUND);
}
A continuación podemos ver cómo quedará la traza del ejemplo anterior registrada en el log de la aplicación:

2021-03-02 21:47:55,813 ERROR [http-nio-8081-exec-2] c.s.m.c.ProductController                       : Product not found: 2
com.sanvalero.myshop.exception.ProductNotFoundException: Product not found: 2
	at com.sanvalero.myshop.service.ProductServiceImpl.lambda$deleteProduct$1(ProductServiceImpl.java:49)
	at java.base/java.util.Optional.orElseThrow(Optional.java:408)
	at com.sanvalero.myshop.service.ProductServiceImpl.deleteProduct(ProductServiceImpl.java:49)
	at com.sanvalero.myshop.controller.ProductController.deleteProduct(ProductController.java:60)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at java.base/jdk.internal.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at java.base/jdk.internal.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.base/java.lang.reflect.Method.invoke(Method.java:566)
	at org.springframework.web.method.support.InvocableHandlerMethod.doInvoke(InvocableHandlerMethod.java:197)
	at org.springframework.web.method.support.InvocableHandlerMethod.invokeForRequest(InvocableHandlerMethod.java:141)
	at org.springframework.web.servlet.mvc.method.annotation.ServletInvocableHandlerMethod.invokeAndHandle(ServletInvocableHandlerMethod.java:106)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.invokeHandlerMethod(RequestMappingHandlerAdapter.java:894)
	at org.springframework.web.servlet.mvc.method.annotation.RequestMappingHandlerAdapter.handleInternal(RequestMappingHandlerAdapter.java:808)
	at org.springframework.web.servlet.mvc.method.AbstractHandlerMethodAdapter.handle(AbstractHandlerMethodAdapter.java:87)
	at org.springframework.web.servlet.DispatcherServlet.doDispatch(DispatcherServlet.java:1060)
	at org.springframework.web.servlet.DispatcherServlet.doService(DispatcherServlet.java:962)
	at org.springframework.web.servlet.FrameworkServlet.processRequest(FrameworkServlet.java:1006)
	at org.springframework.web.servlet.FrameworkServlet.doDelete(FrameworkServlet.java:931)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:658)
	at org.springframework.web.servlet.FrameworkServlet.service(FrameworkServlet.java:883)
	at javax.servlet.http.HttpServlet.service(HttpServlet.java:733)
. . .
. . .
Relaciones entre clases en el modelo de datos
Probar los Servicios Web
Si antes de integrar una aplicación con un determinado servicio web, queremos probar éste para comprobar cómo funcionar, tenemos que usar aplicaciones destinadas para ese propósito, como Postman, que es una aplicación destinada exclusivamente a testear APIs.

Para el servicio web desarrollado a lo largo de este tema, vamos a ver cómo se definirían una serie de pruebas para todos sus endpoints utilizando Postman.

Crearemos una colección y diferentes requests que nos permitan probar todos los endpoints desarrollados en este proyecto (Pincha en la captura para aumentarla y ver cómo configurar cada uno de los casos)


Figure 5: Obtiene todos los productos

Figure 6: Obtiene todos los productos de la misma categoría

Figure 7: Registra un nuevo producto

Figure 8: Modifica un producto existente

Figure 9: Elimina un producto

Figure 10: Devuelve un error porque no existe el producto que se pide eliminar
Ejercicios
Crea una aplicación que ofrezca unos servicios web para la gestión de vuelos. La aplicación tendrá una base de datos de vuelos donde almacenará: origen, destino, precio, numero de escalas y compañia. Deberá ofrecer las siguientes operaciones:
Búsqueda de vuelos, pudiendo filtrar por origen, destino y numero de escalas
Registro de un nuevo vuelo
Dar de baja un vuelo
Dar de baja todos los vuelos a un destino determinado
Modificar un vuelo

Crea una API que ofrezca servicios web de búsqueda de hoteles. Se mantendrá un base de datos de hoteles (nombre, descripción, categoría, ¿piscina?, localidad) y de las habitaciones de los mismos (tamaño, 1 ó 2 personas, precio/noche, ¿incluye desayuno?, ¿ocupada?). Deberá ofrecer, sobre esos datos, las siguientes operaciones:
Búsqueda de hotel por localidad o categoría
Búsqueda de habitaciones de un hotel por tamaño y precio (rango minimo→máximo). Solo mostrará aquellas habitaciones que estén marcadas como libres
Registrar un nuevo hotel
Registrar una nueva habitación a un hotel
Eliminar una habitación determinada de un hotel
Modificar una habitación para indicar que está ocupada