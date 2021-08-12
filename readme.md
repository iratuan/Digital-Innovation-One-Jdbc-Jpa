# Digital Innovation One

## Treinamento sobre Jdbc e JPA



### Introdução

Nesse treinamento iremos aprender um pouco mais sobre o uso de JDBC e JPA, seguindo o guia da Digital Innovation One, com o instrutor **Daniel Karam**

### Parte 1 - Configurando um banco de dados.

Para esse treinamento, utilizaremos o banco de dados h2, um banco de dados em memória e isso irá nos poupar um pouco de tempo.

Nosso projeto é um projeto **maven**, então teremos nossas dependências gerenciadas por ele.

Para iniciar, insira as dependências abaixo no seu arquivo **pom.xml**

```xml
<dependency>
   <groupId>org.hibernate</groupId>
   <artifactId>hibernate-entitymanager</artifactId>
   <version>5.4.27.Final</version>
</dependency>

<dependency>
   <groupId>com.h2database</groupId>
   <artifactId>h2</artifactId>
   <version>1.4.200</version>
</dependency>

<dependency>
   <groupId>junit</groupId>
   <artifactId>junit</artifactId>
   <version>4.13.2</version>
   <scope>test</scope>
</dependency>
```

E adicione o plugin abaixo para podermos utilizar um banco de dados específico para testes.

```xml
<plugin>
   <artifactId>maven-antrun-plugin</artifactId>
   <version>1.3</version>
   <executions>
      <execution>
         <id>copy-test-persistence</id>
         <phase>process-test-resources</phase>
         <configuration>
            <tasks>
               <echo>renaming deployment persistence.xml</echo>
               <move file="${project.build.outputDirectory}/META-INF/persistence.xml" tofile="${project.build.outputDirectory}/META-INF/persistence.xml.proper"/>
               <echo>replacing deployment persistence.xml with test version</echo>
               <copy file="${project.build.testOutputDirectory}/META-INF/persistence-test.xml" tofile="${project.build.outputDirectory}/META-INF/persistence.xml" overwrite="true"/>
            </tasks>
         </configuration>
         <goals>
            <goal>run</goal>
         </goals>
      </execution>
      <execution>
         <id>restore-persistence</id>
         <phase>prepare-package</phase>
         <configuration>
            <tasks>
               <echo>restoring the deployment persistence.xml</echo>
               <move file="${project.build.outputDirectory}/META-INF/persistence.xml.proper" tofile="${project.build.outputDirectory}/META-INF/persistence.xml" overwrite="true"/>
            </tasks>
         </configuration>
         <goals>
            <goal>run</goal>
         </goals>
      </execution>
   </executions>
</plugin>
```

Após isso, crie um arquivo chamado **persistence.xml** dentro de **src/resources/META-INF** com o seguinte conteúdo.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

    <persistence-unit name="treinamento-pu" transaction-type="RESOURCE_LOCAL">
        <class>br.com.loja.model.Produto</class>
        <class>br.com.loja.model.Categoria</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:diojpa"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>

            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

E outro arquivo chamado **persistence-test.xml** dentro de **test/resource/META-INF** que irá apontar para o banco de dados de testes.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

    <persistence-unit name="treinamento-pu" transaction-type="RESOURCE_LOCAL">
        <class>br.com.loja.model.Produto</class>
        <class>br.com.loja.model.Categoria</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:diojpa_test"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>

            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

Note que são bem parecidos, exceto pela linha que mapeia a **url** do banco de dados.

Feito isso, temos o nosso ambiente inicial configurado para começarmos a codificar nosso tutorial.