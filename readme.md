# Curso de persistência com jpa - introdução ao Hibernate



### Repositório do curso Persistência com JPA - Introdução ao Hibernate da Alura.



### Notas importantes:

**JDBC** é a especificação que dita as regras para implementação de comunicação com bancos de dados. Exemplo: quando você baixa um driver **jdbc** para **postgres**, você está baixando uma implementação jdbc específica para o **postgres**, e assim por diante.



- **Hibernate** surgiu como uma alternativa ao **JDBC** e **EJB2 **e foi criado em **2001** por **Gavin King**
- **JPA** é uma especificação para padronização de **ORM** e foi lançada em **2006**
- Além da especificação JPA, você utiliza uma implementação como **Hibernate**, **EclipseLink**, **OpenJPA** que, implementam a especificação.

### Passos importantes

No arquivo **pom.xml** insira as seguintes dependências:

```xml
<dependencies>
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

    <!-- https://mvnrepository.com/artifact/junit/junit -->
    <dependency>
        <groupId>junit</groupId>
        <artifactId>junit</artifactId>
        <version>4.13.2</version>
        <scope>test</scope>
    </dependency>
</dependencies>
```

Note que utilizaremos o H2 como banco de dados. Caso deseje utilizar qualquer outro banco de dados, basta pesquisar no repositório do **maven** as dependências correspondentes.

### Criando o arquivo persistence.xml

Dentro da pasta src/main/resources/META-INF, crie um arquivo chamado persistence.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

    <persistence-unit name="loja-pu" transaction-type="RESOURCE_LOCAL">
        <properties>
            <!-- Particulares da especificação -->
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:loja"/>
            <property name="javax.persistence.jdbc.user" value="sa"/>
            <property name="javax.persistence.jdbc.password" value=""/>
			
            <!-- Particulares do Hibernate -->
            <property name="hibernate.dialect" value="org.hibernate.dialect.H2Dialect"/>
            <property name="hibernate.show_sql" value="true"/>
            <property name="hibernate.format_sql" value="true"/>
            <property name="hibernate.use_sql_comments" value="true"/>
            <property name="hibernate.hbm2ddl.auto" value="update"/>
        </properties>
    </persistence-unit>
</persistence>
```

Note as configurações da especificação e as específicas do hibernate.

### Criando as primeiras entidades

Dentro de **src/main/br/com/loja/models** crie um arquivo chamado Produto.java com o seguinte conteúdo:

```java
package br.com.loja.model;

import javax.persistence.*;
import java.math.BigDecimal;

@Entity
@Table(name ="produtos")
public class Produto {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String nome;
    private String descricao;
    private BigDecimal preco;

    @ManyToOne
    private Categoria categoria;

    public Produto(){

    }

    public Produto(String nome, String descricao, BigDecimal preco, Categoria categoria) 	{
        this.nome = nome;
        this.descricao = descricao;
        this.preco = preco;
        this.categoria = categoria;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }

    public String getDescricao() {
        return descricao;
    }

    public void setDescricao(String descricao) {
        this.descricao = descricao;
    }

    public BigDecimal getPreco() {
        return preco;
    }

    public void setPreco(BigDecimal preco) {
        this.preco = preco;
    }
}

```

```java
package br.com.loja.model;

import javax.persistence.*;

@Entity
@Table(name = "categorias")
public class Categoria {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private long id;
    private String nome;

    public Categoria(){}

    public Categoria(String nome){
        this.nome = nome;
    }

    public long getId() {
        return id;
    }

    public void setId(long id) {
        this.id = id;
    }

    public String getNome() {
        return nome;
    }

    public void setNome(String nome) {
        this.nome = nome;
    }
}

```



Criadas as classes que representam suas entidades no banco de dados, adicione as mesmas no arquivo persistence.xml

```xml
<class>br.com.loja.model.Produto</class>
<class>br.com.loja.model.Categoria</class>
```

Crie agora uma classe generica para encapsular o comportamento básico do DAO

```java
package br.com.loja.dao;

import javax.persistence.*;
import javax.persistence.criteria.CriteriaBuilder;
import javax.persistence.criteria.CriteriaQuery;
import javax.persistence.criteria.Root;
import java.util.List;

public class BaseDAO<T> {
    private EntityManagerFactory emf = Persistence.createEntityManagerFactory("loja-pu");
    private EntityManager em = emf.createEntityManager();
    private Class<T> clazz;

    public EntityManager getEntityManager(){
        return this.em;
    }

    public BaseDAO(Class clazz){
        this.clazz = clazz;
    }

    public T create(T entity) {
        try {
            em.getTransaction().begin();
            em.persist(entity);
            em.getTransaction().commit();
            return entity;
        } catch (PersistenceException e) {
            throw new PersistenceException("Erro ao persistir objeto");
        }
    }

    public List<T> findAll() {
        CriteriaBuilder builder = this.em.getCriteriaBuilder();
        CriteriaQuery<T> cq = builder.createQuery(clazz);
        Root<T> root = cq.from(clazz);
        cq.select(root);
        return this.em.createQuery(cq).getResultList();
    }

    public T getOne(Long id){
        return  em.find(clazz, id);
    }

    public void removeAll() throws Exception {
        List<T> list = this.findAll();
        for (T t : list) {
            em.getTransaction().begin();
            em.remove(t);
            em.getTransaction().commit();
        }
    }

    public T update(T entity) throws Exception {
        try{
            em.getTransaction().begin();
            entity = em.merge(entity);
            em.getTransaction().commit();
            return entity;
        }catch (PersistenceException e){
            throw new PersistenceException("Erro ao persistir objeto");
        }
    }

    public void close(){
        if(em.isOpen()){
            em.close();
        }
        if(emf.isOpen()){
            emf.close();
        }
    }
}

```

Adicione agora as classes que estendem o BaseDAO passando a classe específica

```java
package br.com.loja.dao;

import br.com.loja.model.Produto;

import javax.persistence.Query;
import java.util.List;

public class ProdutoDAO extends BaseDAO<Produto> {
    public ProdutoDAO() {
        super(Produto.class);
    }

    public List<Produto> searchByName(String name) {
        final Query query = getEntityManager().createQuery("Select p from Produto p where p.nome LIKE CONCAT('%',:name,'%')");
        query.setParameter("name", name);
        return query.getResultList();
    }
}

```

```java
package br.com.loja.dao;

import br.com.loja.model.Categoria;

public class CategoriaDAO extends BaseDAO<Categoria> {
    public CategoriaDAO() {
        super(Categoria.class);
    }
}
```

Feito isso, temos o básico para realizar o insert das entidades através de uma classe de teste temporária.

```java
package br.com.loja.testes;

import br.com.loja.dao.CategoriaDAO;
import br.com.loja.dao.ProdutoDAO;
import br.com.loja.model.Categoria;
import br.com.loja.model.Produto;

import java.math.BigDecimal;
import java.util.List;

public class TestaProduto {
    public static void main(String[] args) throws Exception {

        // Cria uma categoria
        Categoria categoria = new Categoria("Celulares");

        // Cria um produto
        Produto celular1 = new Produto("IPhone X1","Celular muito caro", new BigDecimal("12000"), categoria);
        Produto celular2 = new Produto("IPhone X2","Celular muito caro", new BigDecimal("12000"), categoria);
        Produto celular3 = new Produto("IPhone X3","Celular muito caro", new BigDecimal("12000"), categoria);

        // Istanciando os DAOs
        ProdutoDAO produtoDAO = new ProdutoDAO();
        CategoriaDAO categoriaDAO = new CategoriaDAO();

        // Inserido a categoria
        categoriaDAO.create(categoria);
        produtoDAO.create(celular1);
        produtoDAO.create(celular2);
        produtoDAO.create(celular3);

        System.out.println("Quantidade de produtos: " + produtoDAO.findAll().size());
        for (Produto p: produtoDAO.findAll()) {
            System.out.println("Produto " + "ID: "+ p.getId() + " - " + p.getNome());
        }

        System.out.println("Exibindo o produto #1: " + produtoDAO.getOne(1L).getNome());

        produtoDAO.create(celular1);
        Produto produtoRetornado = produtoDAO.getOne(1L);
        produtoRetornado.setNome("Produto atualizado");
        Produto p = produtoDAO.update(produtoRetornado);

        System.out.println("Atualizando um produto: " + p.getNome());

        List<Produto> produtosBuscados = produtoDAO.searchByName("IPhone");
        System.out.println("Pesquisando produtos por nome e retornando " + produtosBuscados.size() + " resultados");

        try {
            produtoDAO.removeAll();
        } catch (Exception e) {
            e.printStackTrace();
        }

        System.out.println("Quantidade de produtos: " + produtoDAO.findAll().size());
    }

}

```

### Criando uma classe de teste unitária para testar o comportamento

```java
package br.com.loja.models;

import br.com.loja.dao.CategoriaDAO;
import br.com.loja.dao.ProdutoDAO;
import br.com.loja.model.Categoria;
import br.com.loja.model.Produto;
import org.junit.Before;
import org.junit.Test;

import java.math.BigDecimal;

import static org.junit.Assert.*;

public class ProdutoTest {

    private Produto produto;
    private Categoria categoria;
    private ProdutoDAO produtoDAO;
    private CategoriaDAO categoriaDAO;

    @Before
    public void init() {
        produtoDAO = new ProdutoDAO();
        categoriaDAO = new CategoriaDAO();
        // Cria uma categoria
        categoria = new Categoria("Celulares");
        // Cria um produto
        produto = new Produto("IPhone X", "Celular muito caro", new BigDecimal("12000"), categoria);
    }

    @Test
    public void deveInserirUmProdutoComCategoria() throws Exception {
        produtoDAO.removeAll();
        categoriaDAO.create(categoria);
        produtoDAO.create(produto);
        assertTrue("A lista de produtos, pós insert, deve ser maior que zero: ", produtoDAO.findAll().size() > 0);
    }

    @Test
    public void deveRetornarUmProduto(){
        categoriaDAO.create(categoria);
        produtoDAO.create(produto);
        assertTrue("Deve resgatar o produto com id #1: " , produtoDAO.getOne(1L).getId() == 1);
    }

    @Test
    public void deveRetornarUmaListaVazia() throws Exception {
        produtoDAO.removeAll();
        assertEquals("Deve retornar uma lista vazia", 0, produtoDAO.findAll().size());
    }

    @Test
    public void deveAtualizarUmProduto() throws Exception {
        categoriaDAO.create(categoria);
        Produto p = produtoDAO.create(produto);
        p.setNome("Produto atualizado");
        p = produtoDAO.update(p);
        assertNotNull("Deve retornar um produto atualizado", p);
        assertTrue("Deve retornar um produto com título atualizado","Produto atualizado".equals(p.getNome()));
    }

    @Test
    public void devePesquisarUmProdutoPorNome(){
        categoriaDAO.create(categoria);
        produtoDAO.create(produto);
        assertTrue("Deve retornar, pelo menos, um produto com o título informado: ", produtoDAO.searchByName(produto.getNome()).size() > 0);
    }
}

```

Precisamos também criar um arquivo persistence-test.xml  dentro de test/resources/META-INF que irá mapear um banco de testes, pois não é interessante utilizar o banco de produção para testar nossas classes.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<persistence version="2.2"
             xmlns="http://xmlns.jcp.org/xml/ns/persistence"
             xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
             xsi:schemaLocation="http://xmlns.jcp.org/xml/ns/persistence http://xmlns.jcp.org/xml/ns/persistence/persistence_2_2.xsd">

    <persistence-unit name="loja-pu" transaction-type="RESOURCE_LOCAL">
        <class>br.com.loja.model.Produto</class>
        <class>br.com.loja.model.Categoria</class>
        <properties>
            <property name="javax.persistence.jdbc.driver" value="org.h2.Driver"/>
            <property name="javax.persistence.jdbc.url" value="jdbc:h2:mem:loja_teste"/>
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

Modifique o arquivo pom.xml com o seguinte códig

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

Ele irá **substituir** o arquivo persistence.xml pelo persistence-test.xml em tempo de execução de testes, garantindo que possamos trabalhar tranquilamente com um banco de produção e outro banco para testes.

Um ponto importante que ressalto é sobre o ciclo de vida de entidade JPA como foi bem lembrado pelo instrutor do curso.

![](https://image.slidesharecdn.com/cefet-2013-04-130408163740-phpapp01/95/mapeamento-objetorelacional-com-java-persistence-api-11-638.jpg?cb=1365439124)





Feito isso, basta executar o testes e temos nosso primeiro teste de integração, testando a inserção das nossas entidades.

Aluno: **@iratuan**

