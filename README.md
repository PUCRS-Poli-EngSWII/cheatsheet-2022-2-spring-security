# Grupo 1

## Spring Boot
## O que é
## Como Funciona Internamente
## Controller Rest
### O que é 
### Anotações e Parâmetros das Anotações
### Exemplos
### Construtores, Acessores e Modificadores Necessários
### Mapeamento de Verbos
### Exemplos
### Como Receber Dados da Chamada
### Como Retornar Dados e Status
### Classes e Métodos Adicionais
### Exemplos
----------------------------------
# Grupo 2

## Testes Unitários e de Integração
### O que são
## A Ferramenta Básica: JUnit
### O que é
### Cookbook
#### Pacotes (imports)
#### Anotações
#### Asserções
#### Execmplos
-----------------------------------
# Grupo 3

## Mocks
### O que são
### Quando usar
## A Ferramenta Mockito
### O que é
### Cookbook
### Pacotes (imports)
### Anotações e cláusulas
### Exemplos
### Testes Unitários e Mocks com Spring Rest Controller
-----------------------------------
# Grupo 4 

## Spring Data JPA
### O que é
## Classes e Interfaces
## Anotações
### Exemplos
## Anotações de Mapeamento
### Exemplos
## Cookbook
### Como usar
### Exemplos
## Testes Unitários e Mocks com Repositórios
### Como fazer
### Exemplos
------------------------------------
# Grupo 5

## Spring Security

## O que é
Spring Security é a *framework* padrão de autenticação e controle de acesso utilizado pelas aplicações Spring.
Ele tem um enfoque principalmente em autenticação e autorização, fornecendo uma camada de proteção contra ataques comuns (como fixação de sessão, *clickjacking*, dentre outros).

## Como funciona
O Spring Security intercepta as requisições recebidas e valida tanto os *payload* quanto os *headers* conforme uma séries de regras definidas pelo desenvolvedor.

Por padrão, ao adicionar o Spring Security ao *classpath*, ele irá exigir autenticação "basic" para todos os *endpoints* HTTP. Para configurar o Spring Security, deve-se criar uma classe anotada com `@Configuration` e `@EnableWebSecurity`. As regras de acesso e autenticação estão todas em `@Bean` dentro desta classe. Por exemplo:

```java
@Configuration
@EnableWebSecurity
public class WebSecurityConfig {

	@Bean
	public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
		http
			.authorizeHttpRequests((requests) -> requests
				.antMatchers("/", "/home").permitAll()
				.anyRequest().authenticated()
			)
			.formLogin((form) -> form
				.loginPage("/login")
				.permitAll()
			)
			.logout((logout) -> logout.permitAll());

		return http.build();
	}
}
```

### Cenários de uso (User details, OAuth com Token)
#### User details

Para proteger as informações do usuário, o Spring Security oferece a interface `UserDetailsService`, a qual contém um único método:

```java
UserDetails loadUserByUsername(String username) throws UsernameNotFoundException
```

Este método não é chamado pela Spring Security diretamente em nenhum lugar, e **só pode recuperar informações de usuários que foram encapsuladas em objetos `Authentication`**. Um exemplo de como usar esta interface seria:

```java
@Service
public class CustomUserDetailService implements UserDetailsService {

    @Autowired
    private CustomerRepository customerRepository;

    @Override
    public UserDetails loadUserByUsername(String username) throws UsernameNotFoundException {
        final CustomerEntity customer = customerRepository.findByEmail(username);
        if (customer == null) {
            throw new UsernameNotFoundException(username);
        }
        UserDetails user = User.withUsername(customer.getEmail()).password(customer.getPassword()).authorities("USER").build();
        return user;
    }
}
```

#### OAuth com Token

OAuth é um protocolo de autorização para que o dono de um recurso permita que terceiros tenham acesso limitado a um serviço HTTP, sem que esses terceiros possam ver a identidade ou credenciais do usuário. Esse tipo de protocolo pode ser usado para, por exemplo, permitir que o *login* de um *website* seja feito através da conta do Google, ou da Microsoft, etc. O usuário efetua o seu login em uma dessas plataformas e nós pedimos permissão para acessar dados e/ou serviços específicos do usuário, que estão associados a estas contas (por exemplo, o Google Drive).
Para usar o protocolo no Spring Security, primeiro precisamos de uma classe de configuração:

```java
@Configuration 
public class UserConfig extends WebSecurityConfigurerAdapter { 
   @Bean 
   public UserDetailsService userDetailsService() {
      UserDetailsManager userDetailsManager = new InMemoryUserDetailsManager(); 
      UserDetails user = User.withUsername("john") 
         .password("12345") .authorities("read") 
      .build(); userDetailsManager.createUser(user); return userDetailsManager; 
   }
   @Bean
   public PasswordEncoder passwordEncoder() {
      return new BCryptPasswordEncoder(); 
   } 
   @Override 
   @Bean 
   public AuthenticationManager authenticationManagerBean() throws Exception { 
      return super.authenticationManagerBean(); 
   } 
}
```

Para usar o protocolo OAuth, precisamos agora configurar a autorização do servidor:

```java
public class AuthorizationServerConfig extends AuthorizationServerConfigurerAdapter {
   @Autowired private AuthenticationManager authenticationManager; 
   @Override 
   public void configure(ClientDetailsServiceConfigurer clients) throws Exception { 
      clients.inMemory() .withClient("oauthclient1") .secret("oauthsecret1") .scopes("read") .authorizedGrantTypes("password") } 
   @Override 
   public void configure(AuthorizationServerEndpointsConfigurer endpoints) throws Exception { 
      endpoints.authenticationManager(authenticationManager); 
   } 
}
```

Estamos usando o tipo de concessão "senha" (`authorizedGrantTypes("password")`, acima). A senha do usuário obviamente deve ser criptografada (não use a classe `NoOpPasswordEncoder`). Para este tipo de autorização, o usuário fornece o seu nome de usuário, senha e escopo da concessão para a aplicação cliente, que então usa essas informações, junto com as suas credencias, para pedir os `tokens` de autorização do servidor de autorização.

### Cadeia de Filtros

Cadeias de filtros são o principal mecanismo usado pela Spring Security para garantir a segurança de uma aplicação. Trata-se de um *pipeline* de validações onde a cada estágio só passam as *requests* que puderem ser validadas.

Todos os `HttpServletRequest` passam pela cadeia de filtros antes de chegar nas controllers. Vários desses filtros já são configurados automaticamente pela Spring. Você pode vê-los ao setar a opção `logging.level.org.springframework.security.web.FilterChainProxy=DEBUG` em `application.properties`. Um *output* possível seria:

```
o.s.security.web.FilterChainProxy        : /home at position 1 of 15 in additional filter chain; firing Filter: 'WebAsyncManagerIntegrationFilter'
o.s.security.web.FilterChainProxy        : /home at position 2 of 15 in additional filter chain; firing Filter: 'SecurityContextPersistenceFilter'
o.s.security.web.FilterChainProxy        : /home at position 3 of 15 in additional filter chain; firing Filter: 'HeaderWriterFilter'
o.s.security.web.FilterChainProxy        : /home at position 4 of 15 in additional filter chain; firing Filter: 'CsrfFilter'
o.s.security.web.FilterChainProxy        : /home at position 5 of 15 in additional filter chain; firing Filter: 'LogoutFilter'
o.s.security.web.FilterChainProxy        : /home at position 6 of 15 in additional filter chain; firing Filter: 'UsernamePasswordAuthenticationFilter'
o.s.security.web.FilterChainProxy        : /home at position 7 of 15 in additional filter chain; firing Filter: 'DefaultLoginPageGeneratingFilter'
o.s.security.web.FilterChainProxy        : /home at position 8 of 15 in additional filter chain; firing Filter: 'DefaultLogoutPageGeneratingFilter'
o.s.security.web.FilterChainProxy        : /home at position 9 of 15 in additional filter chain; firing Filter: 'BasicAuthenticationFilter'
o.s.security.web.FilterChainProxy        : /home at position 10 of 15 in additional filter chain; firing Filter: 'RequestCacheAwareFilter'
o.s.security.web.FilterChainProxy        : /home at position 11 of 15 in additional filter chain; firing Filter: 'SecurityContextHolderAwareRequestFilter'
o.s.security.web.FilterChainProxy        : /home at position 12 of 15 in additional filter chain; firing Filter: 'AnonymousAuthenticationFilter'
o.s.security.web.FilterChainProxy        : /home at position 13 of 15 in additional filter chain; firing Filter: 'SessionManagementFilter'
o.s.security.web.FilterChainProxy        : /home at position 14 of 15 in additional filter chain; firing Filter: 'ExceptionTranslationFilter'
o.s.security.web.FilterChainProxy        : /home at position 15 of 15 in additional filter chain; firing Filter: 'FilterSecurityInterceptor'
```

Podemos verificar que **os filtros são executados em uma ordem específica**. É possível delegar filtros para um outro `bean` usando o `DelegatingFilterProxy` e o `FilterChainProxy`.

É possível definir várias cadeias de filtros, e selecionar qual usar de acordo com o tipo de requisição.

É possível ainda, definir cadeias de filtros diferentes e independentes para aplicações REST, communicações internas e outras aplicações.

Cadeias de filtros são um assunto bastante complexo na Spring Security. Para mais informações, veja a [última versão da documentação oficial](https://docs.spring.io/spring-security/site/docs/5.3.9.RELEASE/reference/html5/).

### Autorização e Roles

A tradução de "*roles*" é "papel", no sentido de "papel de um ator". Dentro de uma sistema complexo empresarial, há várias pessoas com papéis diferentes interagindo com o sistema: administradores, desenvolvedores, usuários, etc. Esses indivíduos não podem ter o mesmo grau de visibilidade para dentro do sistema. Uma maneira de organizar essas permissões seria:

  1. O administrador tem permissão de efetuar qualquer ação
  2. O usuário pode ler e atualizar apenas as informações relevantes a ele
  3. Os desenvolvedores podem ler informaçes do sistema completo, mas cada equipe de desenvolvimento só pode escrever na parte do sistema que lhe compete
 
 Isso significa que cada "papel" que alguém exerce no sistema deve corresponder com uma séria de permissões, ou autorizações (ou ainda, privilégios).

## Cookbook

### Como configurar


### Exemplos
### Autenticação com User Details
### Exemplos
### Autorização (com roles) de RestControllers e Beans
### Exemplos
### Autenticação com OAuth e Token
### Exemplos
### Autorização (com roles) de RestControllers e Beans
### Exemplos
