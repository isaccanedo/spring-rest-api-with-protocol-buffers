## API Spring REST com buffers de protocolo

# 1. Introdução
Protocol Buffers é um mecanismo neutro de linguagem e plataforma para serialização e desserialização de dados estruturados, que é proclamado pelo Google, seu criador, como sendo muito mais rápido, menor e mais simples do que outros tipos de payloads, como XML e JSON.

Este tutorial o orienta na configuração de uma API REST para aproveitar as vantagens dessa estrutura de mensagem baseada em binário.

# 2. Buffers de protocolo
Esta seção fornece algumas informações básicas sobre Buffers de protocolo e como eles são aplicados no ecossistema Java.

## 2.1. Introdução aos buffers de protocolo
Para fazer uso de Buffers de protocolo, precisamos definir estruturas de mensagem em arquivos .proto. Cada arquivo é uma descrição dos dados que podem ser transferidos de um nó para outro ou armazenados em fontes de dados. Aqui está um exemplo de arquivos .proto, que é denominado isaccanedo.proto e reside no diretório src/main/resources. Este arquivo será usado neste tutorial mais tarde:

```
syntax = "proto3";
package isaccanedo;
option java_package = "com.isaccanedo.protobuf";
option java_outer_classname = "isaccanedoTraining";

message Course {
    int32 id = 1;
    string course_name = 2;
    repeated Student student = 3;
}
message Student {
    int32 id = 1;
    string first_name = 2;
    string last_name = 3;
    string email = 4;
    repeated PhoneNumber phone = 5;
    message PhoneNumber {
        string number = 1;
        PhoneType type = 2;
    }
    enum PhoneType {
        MOBILE = 0;
        LANDLINE = 1;
    }
}
```


Neste tutorial, usamos a versão 3 do compilador de buffer de protocolo e da linguagem de buffer de protocolo, portanto, o arquivo .proto deve começar com a sintaxe = declaração “proto3”. Se um compilador versão 2 estiver em uso, essa declaração será omitida. Em seguida, vem a declaração do pacote, que é o namespace para esta estrutura de mensagem para evitar conflitos de nomenclatura com outros projetos.

As duas declarações a seguir são usadas apenas para Java: a opção java_package especifica o pacote para nossas classes geradas viverem e a opção java_outer_classname indica o nome da classe que inclui todos os tipos definidos neste arquivo .proto.

A subseção 2.3 abaixo descreve os elementos restantes e como eles são compilados no código Java.

### 2.2. Buffers de protocolo com Java
Depois que uma estrutura de mensagem é definida, precisamos de um compilador para converter esse conteúdo neutro de linguagem em código Java. Você pode seguir as instruções no repositório de Buffers de protocolo para obter uma versão apropriada do compilador. Como alternativa, você pode baixar um compilador binário pré-compilado do repositório central Maven procurando pelo artefato com.google.protobuf: protoc e, em seguida, escolhendo uma versão apropriada para sua plataforma.

Em seguida, copie o compilador para o diretório src/main do seu projeto e execute o seguinte comando na linha de comando:

```
protoc --java_out=java resources/isaccanedo.proto
```

Isso deve gerar um arquivo de origem para a classe isaccanedoTraining dentro do pacote com.isaccanedo.protobuf, conforme especificado nas declarações de opções do arquivo isaccanedo.proto.

Além do compilador, o tempo de execução dos Buffers de protocolo é necessário. Isso pode ser obtido adicionando a seguinte dependência ao arquivo Maven POM:

```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>3.0.0-beta-3</version>
</dependency>
```

Podemos usar outra versão do runtime, desde que seja igual à versão do compilador. Para obter o mais recente, verifique este link.

### 2.3. Compilando uma descrição de mensagem
Usando um compilador, as mensagens em um arquivo .proto são compiladas em classes Java aninhadas estáticas. No exemplo acima, as mensagens do Curso e do Aluno são convertidas em classes Java do Curso e do Aluno, respectivamente. Ao mesmo tempo, os campos das mensagens são compilados em getters e setters de estilo JavaBeans dentro desses tipos gerados. O marcador, composto por um sinal de igual e um número, no final de cada declaração de campo é a tag única usada para codificar o campo associado na forma binária.

Percorreremos os campos digitados das mensagens para ver como eles são convertidos em métodos de acesso.

Vamos começar com a mensagem do Curso. Possui dois campos simples, incluindo id e course_name. Seus tipos de buffer de protocolo, int32 e string, são convertidos em tipos Java int e String. Aqui estão seus getters associados após a compilação (com as implementações sendo deixadas de fora por questões de brevidade):

```
public int getId();
public java.lang.String getCourseName();
```

Observe que os nomes dos campos digitados devem estar em caixa de cobra (palavras individuais são separadas por caracteres de sublinhado) para manter a cooperação com outros idiomas. O compilador converterá esses nomes em caixa de camelo de acordo com as convenções Java.

O último campo da mensagem do curso, aluno, é do tipo complexo Aluno, que será descrito a seguir. Este campo é anexado pela palavra-chave repetida, o que significa que pode ser repetido qualquer número de vezes. O compilador gera alguns métodos associados ao campo do aluno da seguinte forma (sem implementações):

```
public java.util.List<com.isaccanedo.protobuf.isaccanedoTraining.Student> getStudentList();
public int getStudentCount();
public com.isaccanedo.protobuf.isaccanedoTraining.Student getStudent(int index);
```

Agora passaremos para a mensagem do aluno, que é usada como um tipo complexo da mensagem do campo do aluno do curso. Seus campos simples, incluindo id, first_name, last_name e email são usados para criar métodos de acesso Java:

```
public int getId();
public java.lang.String getFirstName();
public java.lang.String getLastName();
public java.lang.String.getEmail();
```

O último campo, telefone, é do tipo complexo PhoneNumber. Semelhante à mensagem do campo do aluno do curso, este campo é repetitivo e possui vários métodos associados:

```
public java.util.List<com.isaccanedo.protobuf.isaccanedoTraining.Student.PhoneNumber> getPhoneList();
public int getPhoneCount();
public com.isaccanedo.protobuf.isaccanedoTraining.Student.PhoneNumber getPhone(int index);
```

A mensagem PhoneNumber é compilada no tipo aninhado isaccanedoTraining.Student.PhoneNumber, com dois getters correspondentes aos campos da mensagem:

```
public java.lang.String getNumber();
public com.isaccanedo.protobuf.isaccanedoTraining.Student.PhoneType getType();
```

PhoneType, o tipo complexo do campo type da mensagem PhoneNumber, é um tipo de enumeração, que será transformado em um tipo de enum Java aninhado na classe isaccanedoTraining.Student:

```
public enum PhoneType implements com.google.protobuf.ProtocolMessageEnum {
    MOBILE(0),
    LANDLINE(1),
    UNRECOGNIZED(-1),
    ;
    // Other declarations
}
```

# 3. Protobuf na API Spring REST
Esta seção o guiará pela configuração de um serviço REST usando Spring Boot.

### 3.1. Declaração de Feijão
Vamos começar com a definição do nosso @SpringBootApplication principal:

```
@SpringBootApplication
public class Application {
    @Bean
    ProtobufHttpMessageConverter protobufHttpMessageConverter() {
        return new ProtobufHttpMessageConverter();
    }

    @Bean
    public CourseRepository createTestCourses() {
        Map<Integer, Course> courses = new HashMap<>();
        Course course1 = Course.newBuilder()
          .setId(1)
          .setCourseName("REST with Spring")
          .addAllStudent(createTestStudents())
          .build();
        Course course2 = Course.newBuilder()
          .setId(2)
          .setCourseName("Learn Spring Security")
          .addAllStudent(new ArrayList<Student>())
          .build();
        courses.put(course1.getId(), course1);
        courses.put(course2.getId(), course2);
        return new CourseRepository(courses);
    }

    // Other declarations
}
```

O bean ProtobufHttpMessageConverter é usado para converter respostas retornadas por métodos anotados @RequestMapping em mensagens de buffer de protocolo.

O outro bean, CourseRepository, contém alguns dados de teste para nossa API.

O que é importante aqui é que estamos operando com dados específicos do Buffer de protocolo - não com POJOs padrão.

Esta é a implementação simples do CourseRepository:

```
public class CourseRepository {
    Map<Integer, Course> courses;
    
    public CourseRepository (Map<Integer, Course> courses) {
        this.courses = courses;
    }
    
    public Course getCourse(int id) {
        return courses.get(id);
    }
}
```

### 3.2. Configuração do controlador
Podemos definir a classe @Controller para um URL de teste da seguinte maneira:

```
@RestController
public class CourseController {
    @Autowired
    CourseRepository courseRepo;

    @RequestMapping("/courses/{id}")
    Course customer(@PathVariable Integer id) {
        return courseRepo.getCourse(id);
    }
}
```

E novamente - o importante aqui é que o DTO de curso que estamos retornando da camada do controlador não é um POJO padrão. Esse será o gatilho para que ele seja convertido em mensagens de buffer de protocolo antes de ser transferido de volta para o Cliente.

# 4. Clientes REST e testes
Agora que demos uma olhada na implementação simples da API - vamos ilustrar a desserialização das mensagens do buffer de protocolo no lado do cliente - usando dois métodos.

O primeiro aproveita a API RestTemplate com um bean ProtobufHttpMessageConverter pré-configurado para converter mensagens automaticamente.

A segunda é usar o formato protobuf-java para transformar manualmente as respostas do buffer de protocolo em documentos JSON.

Para começar, precisamos configurar o contexto para um teste de integração e instruir o Spring Boot a encontrar informações de configuração na classe Application, declarando uma classe de teste da seguinte maneira:

```
@RunWith(SpringJUnit4ClassRunner.class)
@SpringApplicationConfiguration(classes = Application.class)
@WebIntegrationTest
public class ApplicationTest {
    // Other declarations
}
```

Todos os trechos de código nesta seção serão colocados na classe ApplicationTest.

### 4.1. Resposta Esperada
A primeira etapa para acessar um serviço REST é determinar o URL da solicitação:

```
private static final String COURSE1_URL = "http://localhost:8080/courses/1";
```

Este COURSE1_URL será usado para obter o primeiro curso duplo de teste do serviço REST que criamos antes. Depois que uma solicitação GET é enviada para o URL acima, a resposta correspondente é verificada usando as seguintes afirmações:

```
private void assertResponse(String response) {
    assertThat(response, containsString("id"));
    assertThat(response, containsString("course_name"));
    assertThat(response, containsString("REST with Spring"));
    assertThat(response, containsString("student"));
    assertThat(response, containsString("first_name"));
    assertThat(response, containsString("last_name"));
    assertThat(response, containsString("email"));
    assertThat(response, containsString("john.doe@isaccanedo.com"));
    assertThat(response, containsString("richard.roe@isaccanedo.com"));
    assertThat(response, containsString("jane.doe@isaccanedo.com"));
    assertThat(response, containsString("phone"));
    assertThat(response, containsString("number"));
    assertThat(response, containsString("type"));
}
```

Faremos uso desse método auxiliar em ambos os casos de teste cobertos nas subseções seguintes.

### 4.2. Testando com RestTemplate
Aqui está como criamos um cliente, enviamos uma solicitação GET para o destino designado, recebemos a resposta na forma de mensagens de buffer de protocolo e verificamos usando a API RestTemplate:

```
@Autowired
private RestTemplate restTemplate;

@Teste
public void whenUsingRestTemplate_thenSucceed () {
     ResponseEntity <Course> course = restTemplate.getForEntity (COURSE1_URL, Course.class);
     assertResponse (course.toString ());
}
```


Para fazer esse caso de teste funcionar, precisamos que um bean do tipo RestTemplate seja registrado em uma classe de configuração:

```
@Bean
RestTemplate restTemplate(ProtobufHttpMessageConverter hmc) {
    return new RestTemplate(Arrays.asList(hmc));
}
```

Outro bean do tipo ProtobufHttpMessageConverter também é necessário para transformar automaticamente as mensagens de buffer de protocolo recebidas. Este bean é igual ao definido na subseção 3.1. Como o cliente e o servidor compartilham o mesmo contexto de aplicativo neste tutorial, podemos declarar o bean RestTemplate na classe Application e reutilizar o bean ProtobufHttpMessageConverter.

### 4.3. Testando com HttpClient
A primeira etapa para usar a API HttpClient e converter manualmente as mensagens do buffer de protocolo é adicionar as duas dependências a seguir ao arquivo Maven POM:

```
<dependency>
    <groupId>com.googlecode.protobuf-java-format</groupId>
    <artifactId>protobuf-java-format</artifactId>
    <version>1.4</version>
</dependency>
<dependency>
    <groupId>org.apache.httpcomponents</groupId>
    <artifactId>httpclient</artifactId>
    <version>4.5.2</version>
</dependency>
```

Para as versões mais recentes dessas dependências, dê uma olhada nos artefatos protobuf-java-format e httpclient no repositório central Maven.

Vamos prosseguir para criar um cliente, executar uma solicitação GET e converter a resposta associada em uma instância InputStream usando o URL fornecido:

```
private InputStream executeHttpRequest(String url) throws IOException {
    CloseableHttpClient httpClient = HttpClients.createDefault();
    HttpGet request = new HttpGet(url);
    HttpResponse httpResponse = httpClient.execute(request);
    return httpResponse.getEntity().getContent();
}
```

Agora, converteremos as mensagens do buffer de protocolo na forma de um objeto InputStream em um documento JSON:

```
private String convertProtobufMessageStreamToJsonString(InputStream protobufStream) throws IOException {
    JsonFormat jsonFormat = new JsonFormat();
    Course course = Course.parseFrom(protobufStream);
    return jsonFormat.printToString(course);
}
```

E aqui está como um caso de teste usa métodos auxiliares privados declarados acima e valida a resposta:

```
@Test
public void whenUsingHttpClient_thenSucceed() throws IOException {
    InputStream responseStream = executeHttpRequest(COURSE1_URL);
    String jsonOutput = convertProtobufMessageStreamToJsonString(responseStream);
    assertResponse(jsonOutput);
}
```

### 4.4. Resposta em JSON
Para deixar claro, os formulários JSON das respostas que recebemos nos testes descritos nas subseções anteriores estão incluídos aqui:


```
id: 1
course_name: "REST with Spring"
student {
    id: 1
    first_name: "John"
    last_name: "Doe"
    email: "john.doe@isaccanedo.com"
    phone {
        number: "123456"
    }
}
student {
    id: 2
    first_name: "Richard"
    last_name: "Roe"
    email: "richard.roe@isaccanedo.com"
    phone {
        number: "234567"
        type: LANDLINE
    }
}
student {
    id: 3
    first_name: "Jane"
    last_name: "Doe"
    email: "jane.doe@isaccanedo.com"
    phone {
        number: "345678"
    }
    phone {
        number: "456789"
        type: LANDLINE
    }
}
```

# 5. Conclusão
Este tutorial apresentou rapidamente os buffers de protocolo e ilustrou a configuração de uma API REST usando o formato com Spring. Em seguida, passamos para o suporte ao cliente e o mecanismo de desserialização de serialização.