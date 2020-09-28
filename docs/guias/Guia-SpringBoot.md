# Guia SpringBoot
## Introducción

La capa Intermedia, comúnmente también conocida como Back-End debe orientarse a la presentación de servicios web del tipo RESTFul entregando información hacia la capa de presentación. Esta capa intermedia se encarga de la lógica de negocios y el acceso a datos y debe ser pensada para que pueda ser implementada como un micro servicio que pueda corren en contenedores tanto en la nube como On Premise.

En general, esta capa puede ser desarrollada en cualquier lenguaje, sin embargo, el presente Documento aborda el desarrollo con Tecnología Spring Boot Java.
## Antecedentes

El framework Spring Boot es una tecnología Java basada en Spring, presentando una infraestructura ligera y que permite simplificar el trabajo de configuración de las aplicaciones.

Podemos resumir en tres fases el desarrollo con Spring Boot:
Seleccionar las dependencias o los jar a través de maven
Crear la aplicación
desplegar en el servidor

Spring Boot nace con la intención de simplificar los pasos 1 y 3 antes mencionados.


## Pre-requisitos

1. VERSIÓN DE SPRING BOOT:
La institución deberá usar la versión Spring Boot 2.2.X, para el desarrollos de sus microservicios.
La versión de Spring Boot queda definida la sección “parent” en el archivo pom.xml del proyecto:

```
	<parent>
		<groupId>org.springframework.boot</groupId>
		<artifactId>spring-boot-starter-parent</artifactId>
		<version>2.2.X.RELEASE</version>
		<relativePath/> <!-- lookup parent from repository -->
	</parent>
```
2. COMPILACIÓN DE LAS FUENTES
Al ser Spring Boot una tecnología java integrada a maven o gradle para el manejo de dependencias y como la institución se está moviendo hacia la práctica IC/DC (Integración/distribución continuas), es requerimiento que las aplicaciones desarrolladas con este framework sean compilables a través de la línea de comando, asegurando así que no haya dependencias del IDE en el que de desarrolló.
Entonce la aplicación se debe poder compilar satisfactoriamente con el comando:

`mvn clean compile package `

3. DEPENDENCIAS: REPOSITORIO DE ARTEFACTOS
Todas las dependencias (artefactos jar, war, ear) de la aplicación deben poder ser descargadas de uno de los siguientes repositorios:

maven central : repositorio público de artefactos de terceros ajenos a la institución
tesorería: repositorio privado designado para las aplicaciones desarrolladas en o para la institución ubicado en la siguiente url:

`http://ec2-3-229-82-198.compute-1.amazonaws.com:8081/artifactory/tesoreria-releases`

4. ARTEFACTOS GENERADOS: REPOSITORIO DE ARTEFACTOS
Todos los artefactos generados y correctamente versionados deben ser subidos al repositorio de artefactos para quedar disponibles para otras aplicaciones, especialmente aquellos artefactos librerías de uso común desarrollas o de las cuales la institución es propietaria.

5. VERSIONAMIENTO:
Las versiones de las ramas de desarrollo deben se nombradas con el número de versión más el postfijo -SNAPSHOT, palabra reservada del sistema de dependencias maven para identificar versiones no productivas.

Ejemplo, archivo pom.xml: 
```xml
<groupId>liquidacioncut.monex</groupId>
	<artifactId>api_LiquidacionCutMonex</artifactId>
	<version>0.0.1-SNAPSHOT</version>
	<name>api_LiquidacionCutMonex</name>
	<description>API impuesto digital liquidacion cut monex</description>
```
Una vez que se libera el desarrollo a ambientes de TEST se debe quitar la palabra clave SNAPSHOT

Ejemplo: 
```xml
<groupId>liquidacioncut.monex</groupId>
	<artifactId>api_LiquidacionCutMonex</artifactId>
	<version>0.0.1</version>
	<name>api_LiquidacionCutMonex</name>
	<description>API impuesto digital liquidacion cut monex</description>
```
6. CÓDIGO FUENTE:
Todos los códigos fuentes deben ser codificados en UTF-8.

7. TIEMPOS DE RESPUESTA PARA API: 
Para el desarrollo de API RESTFul los tiempos de respuesta no deben superar los 3 segundos.

8. Desarrollo de API RESTful:
Si utiliza la tecnología Spring Boot para el desarrollo de APIs RESTful, se debe ajustar la especificación de “Estándares API RESTful” de la institución, la que pude encontrar en el siguiente link:

`https://drive.google.com/file/d/1mmbIFq-DnzNJSDy-CIA4IqEd5dGlNaRz/view?usp=sharing`

## Desarrollo
1. Para iniciar un nuevo proyecto créelo a partir de las plantillas generadas 
a través del sitio https://start.spring.io/

2. Se recomienda separar su aplicación en al menos los siguientes proyectos recomendados:

  * Capa WEB: si su proyecto va a tener una interfaz Web de interacción de con el usuario, deje esta parte en un proyecto WEB separado exclusivo e independiente solo para estos efectos, típicamente un WAR, e implemente solo los clientes necesarios para acceder a las capas de Servicios o APIs que realicen las lógicas de negocios y persistencias. Esto permitirá que esta parte sea desplegada en un sistema de contenedores apartes de la capa de servicios. Aquí, agregue siempre como dependencias los JARs de definición de servicios.

  * Capa de Servicios: si su proyecto va realizar una implementación de una API Rest, cree un proyecto a parte, típicamente un WAR,  de que se encargue exclusivamente de esa capa y que solo se preocupe del mapeo y  exposición de de los servicios definido en la capa de definición de servicios. Aquí agregue siempre como dependencias los JARs de definición de servicios.

  * Capa de definición de servicios: cree un proyecto aparte, típicamente un JAR, donde se definan las interfaces, excepciones y VOs que deben ser implementados para cumplir con los servicios que requieran las capas WEB/API de su aplicación. Estos JARs serán dependencia de todas las otras partes de su aplicación

  * Capa de implementación: como la capa de definición sólo define las interfaces o el que se debe implementar para que las capas WEB/API funcionen apropiadamente y como las implementaciones pueden ser diversas, cree proyectos de implementación separados, típicamente JARs, para cada una de las posibles formas en que pueden ser solucionadas las interfaces definidas en la capa de definición de servicios. en estos proyectos debe agregar siempre como dependencias los jars de definición de servicios.
 
3. Si realiza un desarrollo destinado a correr en un ambiente Cloud: las clases de su aplicación deben estar dentro del paquete :
```
cl.tgr.<negocio>.<sistema>.<función>
Ejemplos:
cl.tgr.arquitectura.resgistroeventosapi.controller
cl.tgr.racaudacion.notificaciondepago.service
```
dónde función puede ser una de las siguientes, sin perjuicio de que puede crear las que estime necesarias:
  * service: paquete donde se deben poner las clases de servicio que son las clases intermediarias entre el DAO y el controlador. Aquí se debe crear al menos una interfaz y para cada interfaz al menos una clase que implemente las interfaces.
  * config: paquete donde se deben poner las clases que contienen configuraciones de Spring Boot por ejemplo  configuraciones de datasources de bases de datos o configuraciones de la librería Jersey
  * controller: paquete donde se deben poner las clases controladoras de los requerimientos de los usuarios, gestionará las peticiones de los usuarios que se hagan a nuestra API o a nuestro sitio WEB respectivamente.
  * mapper: paquete donde se deben poner las clases de mapeos de acceso a datos en caso de utilizar JPA,
repository: para la capa que administra repositorios en caso de usar JPA
  * client: paquete donde se deben poner las clase de clientes de otros sistemas. Si usa un cliente http las clases que se implementan deben ir aquí.
  * exception: paquete donde se pondrán las clases exception creadas para el mejor manejo de errores de la aplicación.
  * vo: paquete donde se pondrán las clases vo o Value Object que utilices su aplicación.
  * dao: paquete donde se pondrán todas las clases que realicen consultas a base de datos.
  * entity: paquete donde se pondrán las clases entidad utilizadas por JPA
  * util: paquete donde se pondrán otras clases de apoyo utilizadas por la aplicación
  * locator: Utilice este patrón de diseño para ubicar recursos JNDI que requiera utilizar en su aplicación. 
  * delegate: Utilice este patrón de diseño para exponer hacia el exterior sus métodos java y que hacia el interior son delegados a clases asociadas especializadas.
 
4. Clase principal: la clase principal de Spring Boot, que contiene la función main y la anotación @SringBootApplication, debe estar en el nivel principal de la aplicación a nivel de 
```
cl.tgr.<negocio>.<sistema>
Ejemplos:
cl.tgr.arquitectura.resgistroeventosapi:
	RegistroEventosApiApplication
cl.tgr.racaudacion.notificaciondepago:
	NotificacionPagosApiApplication
```

5. Si realiza un desarrollo destinado a correr en un ambiente On Premise: las clases de su aplicación deben estar dentro del paquete:
`cl.tesoreria.<negocio>.<sistema>.<subpaquete>.<función>`
donde la función puede ser una de las nombradas en el letra b) anterior.

6. EJB: Spring Boot está pensado para el desarrollo de micro-servicios, que preferente correrán en un Servidor Tomcat el cual no soporta la implementación de EJB. Como tal el servicio entero se replicará por lo que se hace redundante la utilización de EJBs.

7. Para el desarrollo de Micro-Servicios RESTFul,  Spring Boot se debe configurar para usar Tomcat embebido en formato WAR.

8. Para el creación de simples librerías que pueden serán utilizados en otros desarrollos debe seleccionar el formato JAR.

9. Las aplicaciones que implementan microservicios deben ser pensados para mantener la atomicidad de la función que cumplen, es decir, un microservicio debe entregar un servicio de negocio único y en lo posible desacoplado de otros servicios.

10. Estructura de directorios:
La estructura de directorios o carpetas utilizada por Spring Boot responde primeramente a la estructura de directorios maven y luego a su estructura típica propia.

11. Uso de Logger: utilice el propio de Spring Boot para sus logger, recuerde establecerlo en debug para sus depuraciones y en info solo para aquello que sea relevante en producción.

Instanciación, ejemplo:

`private static final Logger logger = LoggerFactory.getLogger(RegistroEventosApplication.class);`

Configuración en application.propierties :

```
## configuracion de Logger General 
logging.level.root=INFO
logging.level.org.springframework=INFO

##Configuración del Logger Específico de la API (por clases)
logging.level.cl.tesoreria.arquitectura.registroeventosapi=DEBUG

#output to a temp_folder/file, archivo de salida 
#logging.file=${java.io.tmpdir}/application.log
logging.file=application.log

# Logging pattern for the console, patron de salida del logger para consola
#logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.pattern.console=%d{HH:mm:ss.SSS} [%thread] %-5level  %class{36}.%M %L  - %msg%n

# Logging pattern for file, patron de salida del logger par el archivo
#logging.pattern.file= %d{yyyy-MM-dd HH:mm:ss} [%thread] %-5level %logger{36} - %msg%
logging.pattern.file= %d{HH:mm:ss.SSS} [%thread] %-5level  %class{36}.%M %L  - %msg%n

```
12. Manejo de Excepciones: Se recomienda que su aplicación implemente su propio sistema de control de excepciones, siempre en el nivel de la definición de los servicios.

13. Manejo de propiedades: Se recomienda usar etiquetas de propiedades dentro del archivo application.properties propio de Srping Boot, típicamente aquellas que pueden variar de ambiente en ambiente, como conexiones a bases de datos, End Points de Servicios Web, nombre de recursos JNDI entro otros.
Puede obtener estas etiquetas y sus valores desde el “controller” a través el contexto Spring Boot, definiendo una variable privada a nivel de clase y autorellenada, ejemplo:

```
@Autowired
	private Environment env;

```


Luego puede definir sus servicios para que los métodos expuestos reciban esta variable, ejemplo :
```
@Service
public interface AlgunService {

	public String metodo(Tipo parametro, Tipo2 parametro2, … ,Environment env) throws ServiceException;

```
y finalmente obtener los valores necesarios, ejemplo :

```
env.getProperty("implementation.date-format")
```
Esto permite centralizar la configuración de su aplicación en un solo archivos lo que facilita la distribución y despliegue de la misma en distintos ambientes.

14. Conexiones a bases de datos: deben usarse los Datasources Externalizados de Spring Boot, los que se configuran en el archivo application.properties.

	Ejemplo sección de configuración de datasources :

```
#Settings for datasource connection to oracle1
oracle1.datasource.url=jdbc:oracle:thin:@192.168.7.150:1521/teso
oracle1.datasource.username=chageuser
oracle1.datasource.password=changepassword
oracle1.datasource.driver-class-name=com.mysql.jdbc.Driver
 
#Settings for database connection to oracle2
oracle2.datasource.url=jdbc:oracle:thin:@192.168.7.150:1521/teso
oracle2.datasource.username=changeuser
oracle2.datasource.password=changepassword
oracle2.datasource.driver-class-name=com.mysql.jdbc.Driver
```

Se debe luego crear una clase java de configuración de los datasources, DatabaseConfig.java :

```java
package mypackage;

import javax.persistence.EntityManagerFactory;
import javax.sql.DataSource;

import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.boot.autoconfigure.jdbc.DataSourceProperties;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.boot.orm.jpa.EntityManagerFactoryBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Primary;
import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
import org.springframework.orm.jpa.JpaTransactionManager;
import org.springframework.orm.jpa.LocalContainerEntityManagerFactoryBean;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.annotation.EnableTransactionManagement;

import com.zaxxer.hikari.HikariDataSource;


@Configuration
public class DatabaseConfig {

@Primary
@Bean(name = "oracle1DataSourceProperties")
	@ConfigurationProperties("oracle1.datasource")
	
public DataSourceProperties dataSourcePropertiesOracle1() {
		return new DataSourceProperties();	
}

@Primary
@Bean(name = "oracle1DataSource")
@ConfigurationProperties(“oracle1.datasource.configuration")
	
public DataSource getDataSourceOracle1(
@Qualifier("oracle1DataSourceProperties")    DataSourceProperties oracle1DataSourceProperties) {
return    oracle1DataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class)
				.build();
	}

@Bean(name = "oracle2DataSourceProperties")
	@ConfigurationProperties("oracle2.datasource")
	
public DataSourceProperties dataSourcePropertiesOracle2() {
		return new DataSourceProperties();	
}

@Bean(name = "oracle2DataSource")
@ConfigurationProperties(“oracle2.datasource.configuration")
	
public DataSource getDataSourceOracle2(
@Qualifier("oracle2DataSourceProperties")    DataSourceProperties oracle1DataSourceProperties) {
return    oracle2DataSourceProperties.initializeDataSourceBuilder().type(HikariDataSource.class)
				.build();
	}

}

```

Finalmente (uso):


```java
DatabaseConfig databaseConfig = new DatabaseConfig()
Datasource oracle1DS = databaseConfig.getDatasourceOracle1()
```




