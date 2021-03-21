# Implantar um Spring Boot WAR em um servidor Tomcat

# 1. Introdução
Spring Boot é uma convenção sobre estrutura de configuração que nos permite definir uma configuração pronta para produção de um projeto Spring, e Tomcat é um dos mais populares Java Servlet Containers.

Por padrão, Spring Boot constrói um aplicativo Java autônomo que pode ser executado como um aplicativo de desktop ou ser configurado como um serviço do sistema, mas há ambientes onde não podemos instalar um novo serviço ou executar o aplicativo manualmente.

Ao contrário dos aplicativos autônomos, o Tomcat é instalado como um serviço que pode gerenciar vários aplicativos dentro do mesmo processo aplicativo, evitando a necessidade de uma configuração específica para cada aplicativo.

Neste guia, vamos criar um aplicativo Spring Boot simples e adaptá-lo para funcionar dentro do Tomcat.

# 2. Configurando um aplicativo Spring Boot
Vamos configurar um aplicativo da web Spring Boot simples usando um dos modelos iniciais disponíveis:

```
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId> 
    <version>2.4.0</version> 
    <relativePath/> 
</parent> 
<dependencies>
    <dependency> 
        <groupId>org.springframework.boot</groupId> 
        <artifactId>spring-boot-starter-web</artifactId> 
    </dependency> 
</dependencies>
```

Não há necessidade de configurações adicionais além do @SpringBootApplication padrão, pois o Spring Boot cuida da configuração padrão.

Adicionamos um EndPoint REST simples para retornar algum conteúdo válido para nós:


```
@RestController
public class TomcatController {

    @GetMapping("/hello")
    public Collection<String> sayHello() {
        return IntStream.range(0, 10)
          .mapToObj(i -> "Hello number " + i)
          .collect(Collectors.toList());
    }
}
```

Agora vamos executar o aplicativo com mvn spring-boot: execute e inicie um navegador em 
http://localhost:8080/hello para verificar os resultados.

# 3. Criação de um Spring Boot WAR
Os contêineres de servlet esperam que os aplicativos atendam a alguns contratos a serem implantados. Para Tomcat, o contrato é o Servlet API 3.0.

Para que nosso aplicativo cumpra este contrato, temos que realizar algumas pequenas modificações no código-fonte.

Primeiro, precisamos empacotar um aplicativo WAR em vez de um JAR. Para isso, alteramos o pom.xml com o seguinte conteúdo:

```
<packaging>war</packaging>
```

Agora, vamos modificar o nome do arquivo WAR final para evitar a inclusão de números de versão:

```
<build>
    <finalName>${artifactId}</finalName>
    ... 
</build>
```

Em seguida, vamos adicionar a dependência do Tomcat:

```
<dependency>
   <groupId>org.springframework.boot</groupId>
   <artifactId>spring-boot-starter-tomcat</artifactId>
   <scope>provided</scope>
</dependency>
```

Por fim, inicializamos o contexto do Servlet exigido pelo Tomcat implementando a interface SpringBootServletInitializer:

```
@SpringBootApplication
public class SpringBootTomcatApplication extends SpringBootServletInitializer {
}
```

Para construir nosso aplicativo WAR implantável por Tomcat, executamos o pacote mvn clean. Depois disso, nosso arquivo WAR é gerado em target/spring-boot-tomcat.war (assumindo que o Maven artifactId é “spring-boot-tomcat”).

Devemos considerar que esta nova configuração torna nosso aplicativo Spring Boot um aplicativo não autônomo (se você gostaria de tê-lo funcionando em modo autônomo novamente, remova o escopo fornecido da dependência do tomcat).

# 4. Implementando o WAR no Tomcat
Para ter nosso arquivo WAR implantado e em execução no Tomcat, precisamos concluir as seguintes etapas:

- Baixe o Apache Tomcat e descompacte-o em uma pasta do tomcat;
- Copie nosso arquivo WAR de target/spring-boot-tomcat.war para a pasta tomcat/webapps/;
- De um terminal, navegue até a pasta tomcat/bin e execute;
- - catalina.bat executado (no Windows);
- - catalina.sh run (em sistemas baseados em Unix);
- Vá para http://localhost:8080/spring-boot-tomcat/hello.

Esta foi uma configuração rápida do Tomcat, verifique o guia de instalação do Tomcat para um guia de configuração completo. Existem também maneiras adicionais de implementar um arquivo WAR no Tomcat.

# 5. Conclusão

Neste breve guia, criamos um aplicativo Spring Boot simples e o transformamos em um aplicativo WAR válido implementável em um servidor Tomcat.