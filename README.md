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

#### OAuth com Token

### Cadeia de Filtros


### Autorização e Roles


## Cookbook

### Como configurar


### Exemplos
### Autenticação com User Details
### Exemplos
### Autorização (com roles) de RestControllers e Beans
### Exemplos
### Autenticação com OAuth e Toke
### Exemplos
### Autorização (com roles) de RestControllers e Beans
### Exemplos
