# Especificação da Konduto

A Konduto é uma linguagem escrita em formato JSON. Com o intuito de facilitar a leitura por humanos, o schema será apresentado em linguagem Typescript.

Alguns itens do JSON correspondem a componentes gráficos (campos de texto, botões, combos, etc) que serão exibidos na tela, e alguns desses componentes disparam eventos.

O que o sistema deve fazer quando esses eventos são disparados são as chamadas ações, as quais são descritas na seção 2.2.

## 1. Estrutura JSON

A seção principal indica os dados principais dos sistema:

- Nome
- Modelo de Entidades e Relacionamento
- Funcionalidades

```typescript
interface Sistema {
	nome: string;
	entidades: Entidade[];
	funcionalidades: Funcionalidade[];
}
```

### 1.1. Modelo de Entidades e Relacionamentos

A seção Entidade tem a seguinte estrutura:

```typescript
interface Entidade {
	nome: string;
	atributos: Dado[];
}

interface Dado {
	nome: string;
	múltiplo?: boolean; // se ausente, considera que é false
	tipo?: string;
	entidade?: string;
}
```

Note que um Modelo de Entidades e Relacionamentos deveria conter as entidades e os relacionamentos, mas a Konduto registra apenas as entidades. Portanto, é necessário explicar como a Konduto especifica os relacionamentos.

Quando uma entidade não possui nenhum relacionamento, a estrutura Entidade vai conter os atributos da entidade em questão. Por exemplo, uma entidade Pessoa poderia ser representada dessa forma:
```typescript
{
	“nome”: “Pessoa”;
	“atributos”: [
		{
			“nome”: “Nome”,
			"tipo”: “Texto”
		},
		{
			“nome”: “Idade”,
			“tipo”: “Número"
		}
	]
}
```
Quando o relacionamento é 1x1, ele será representado de forma análoga às Chaves Estrangeiras de banco de dados: escolhe-se uma das entidades para referenciar a outra. Por exemplo, se o sistema possuir uma entidade Documento, podemos representá-la assim:
```typescript
{
	“nome”: “Documento”;
	“atributos”: [
		{
			“nome”: “Cidadão”,
			“tipo”: “Pessoa"
		},
		{
			“nome”: “Número”,
			“tipo”: “Texto"
		},
		{
			“nome”: “Órgão”,
			“tipo”: “Texto"
		}
	]
}
```
Quando o relacionamento é 1xN, ele será representado de forma análoga às Chaves Estrangeiras de banco de dados: a entidade com cardinalidade N referencia a outra. Por exemplo, se o sistema possuir uma entidade Propriedade, podemos representá-la assim:
```typescript
{
	“nome”: “Propriedade”;
	“atributos”: [
		{
			nome: “Dono”,
			tipo: “Pessoa"
		},
		{
			nome: “Nome”,
			tipo: “Texto"
		},
		{
			nome: “Tipo”,
			tipo: “Texto"
		}
	]
}
```
Quando o relacionamento é NxN, é necessário registrar na Konduto uma entidade correspondente ao relacionamento entre as entidades originais, mesmo que o relacionamento em si não possua nenhum atributo próprio.. Por exemplo, se o sistema precisa registrar quem vendeu o que e a quem, podemos representar assim:

```typescript
{
	“nome”: “Venda”;
	“atributos”: [
		{
			“nome”: “Vendedor”,
			“tipo”: “Pessoa"
		},
		{
			“nome”: “Comprador”,
			“tipo”: “Pessoa"
		},
		{
			“nome”: “Produto”,
			“tipo”: “Propriedade"
		}
	]
}
```

### 1.2. Funcionalidades

Para Konduto, uma funcionalidade tem os seguintes dados:

```typescript 
interface Funcionalidade {
	nome: string;
	página: boolean,
	url?: string;
	modulo?: string;
	entidade: string;
	variáveis?: Dado[];
	parâmetros?: Dado[];
	tipos?: Entidade[];
	classe?: string;
	estilo?: string;
	componentes?: Componente[];
	aoIniciar: string; // ações Konduto
}
```

#### 1.2.1. Nome

O atributo “nome” indica o nome da funcionalidade, e é por ele que ela será referenciada. Ele pode ser usado para gerar a URL, para mudar de página (ação Direcionar seção x.y.z TODO), para gerar requisitos, etc.

#### 1.2.2. Página

O atributo “página” indica se a funcionalidade está associada a uma página própria, ou seja, ao ser executada, vai mudar a URL do navegador e remover todos os componentes da tela atual, perder os dados salvos, enfim, todo o contexto é perdido.

Caso a funcionalidade atual precise transmitir dados à seguinte, isto deve ser feito via parâmetros, conforme seção 2.1.2.7.

#### 1.2.3. URL

O atributo “url” indica a URL da página, e, portanto, deve ser ignorado caso “pagina” seja false. Caso “pagina” seja true e o atributo “url” esteja ausente, a url deve ser gerada automaticamente. 

#### 1.2.4. Módulo

Em sistemas grandes, é comum agrupar as funcionalidades em módulos. O atributo “módulo” indica o módulo da funcionalidade. Caso esteja ausente, supõe que a funcionalidade está no módulo default.

#### 1.2.5. Entidade

Geralmente uma funcionalidade está claramente associada a uma entidade específica.

Por exemplo, “Cadastrar Pessoa” está fortemente relacionada à entidade “Pessoa”.

O atributo “entidade” indica esta entidade.

Este atributo é obrigatório por uma questão técnica de geração de código: no backend, os arquivos “Resource” costumam estar associados às entidades (exemplo, “PessoaResource”). Esse atributo, portanto, indica ao gerador qual Resource deve ser utilizado.

#### 1.2.6. Variáveis

Em alguns casos, o comportamento especificado depende de dados temporários.

Por exemplo, se o sistema deve exibir uma mensagem após o usuário clicar 3 vezes num botão, então, independente da linguagem de programação utilizada, será necessário criar uma variável para armazenar quantas vezes o usuário já clicou no botão.

Dados como esse contador serão declarados no atributo “variáveis”. Caso este atributo esteja ausente, deve-se considerar que ele é uma lista vazia.

#### 1.2.7. Parâmetros

Algumas funcionalidades precisam receber dados para serem executadas. 

Por exemplo, “Remover Pessoa” precisa receber no mínimo o ID da pessoa a ser removida.

Esses dados são declarados no atributo “parâmetros”. Caso este atributo esteja ausente, deve-se considerar que ele é uma lista vazia.

#### 1.2.8. Tipos

Às vezes é preciso declarar certas estruturas que só serão utilizadas por uma determinada funcionalidade. Essas estruturas podem surgir de diversas necessidades, tais como manipulação de dados ou exibição correta em componentes gráficos.

É no atributo “tipos” que essas estruturas são declaradas. Caso este atributo esteja ausente, deve-se considerar que ele é uma lista vazia.

Este atributo deve conter apenas dados compostos. Para declarar dados primitivos, deve-se utilizar o atributo “variáveis”, conforme seção 2.1.2.6.

#### 1.2.9. Classe e Estilo

O conteúdo dos atributos “classe” e “estilo” correspondem, respectivamente, ao conteúdo exato dos atributos “class” e “style” do componente container da funcionalidade, seja ela uma página ou uma modal.

Esses atributos são úteis para o gerenciamento de layout, ou seja, eles vão indicar como os componentes serão dispostos visualmente na tela.

#### 1.2.10. Ações

O atributo “aoIniciar” contém o texto que descreve as ações que devem ser executadas assim que a funcionalidade é acionada, e, portanto, deve obedecer a sintaxe descrita na seção 2.2.

Essas ações só podem referenciar os dados (componentes, variáveis, parâmetros, etc) da própria funcionalidade.

#### 1.2.11. Componentes

### 1.3. Estrutura do JSON Completa

```typescript
interface Sistema {
	nome: string;
	entidades: Entidade[];
	funcionalidades: Funcionalidade[];
}

interface Entidade {
	nome: string;
	atributos: Dado[];
}

interface Dado {
	nome: string;
	múltiplo?: boolean;
	tipo?: string;
	entidade?: string;
}

interface Funcionalidade {
	nome: string;
	módulo?: string;
	entidade: string;
	componentes?: Componente[];
	variáveis?: Dado[];
	parâmetros?: Dado[];
	tipos?: Entidade[];
	aoIniciar: string; // ações Konduto
}

interface Componente {
	botão: Botão;
	campoTextoSimples: CampoTextoSimples;
	campoTextoGrande: CampoTextoGrande;
	campoTextoMascara: CampoTextoMascara;
	campoSomenteLeitura: CampoSomenteLeitura;
	tabela: Tabela;
	modal: Modal;
	comboBox: ComboBox;
	filler: Filler;
}

interface Botão {
	nome: string;
	texto?: string;
	icone?: string;
	aoClicar?: string; // ações Konduto
	largura: number;
}

interface CampoSomenteLeitura {
	nome: string;
	largura: number;
}

interface CampoTextoSimples {
	nome: string;
	largura: number;
}

interface CampoTextoGrande {
	nome: string;
	largura: number;
}

interface CampoTextoMascara {
	nome: string;
	largura: number;
	mascara: string;
}

interface Tabela {
	nome: string;
	entidade: string;
	largura: number;
	colunas: Coluna[];
}

interface Coluna {
	nome: string;
	componente: Componente;
}

interface Modal {
	nome: string;
	titulo: string;
	largura: number;
	componentes: Componente[];
	aoAbrir: string; // ações Konduto
}

interface ComboBox {
	nome: string;
	largura: number;
	aoInicializar: string;
	aoSelecionar: string; // ações Konduto
}

interface Filler {
	nome: string;
	largura: number;
}
```

## 2. Ações

Nesta seção são apresentadas as ações permitidas pela Konduto.

### 2.1. Exibe Componentes

Exibe na tela os componentes informados. Os componentes devem ser referenciados pelos nomes. A tela pode ser a tela principal ou uma modal.
Ao iniciar uma funcionalidade, todos os componentes devem estar ocultos. Portanto, um componente só é exibido após esta ação ser executada.

Sintaxe:
```
<acao_exibe_campos> ::= “exibe campos” <lista_nomes> “;”
<lista_nomes> ::= <referencia_campo> | <referencia_campo> “,” <lista_nomes>
<referencia_campo> ::= “‘” <nome_campo> “‘”
```
Exemplos:
```
exibe campos ‘Nome’;
exibe campos ‘Nome’,’CPF’,’Salvar’;
```
### 2.2. Omite Componentes

Omite os componentes informados. Os componentes devem ser referenciados pelos nomes.

Sintaxe:
```
<acao_omite_campos> ::= “omite campos” <lista_nomes> “;”
<lista_nomes> ::= <referencia_campo> | <referencia_campo> “,” <lista_nomes>
<referencia_campo> ::= “‘” <nome_campo> “‘”
```
Exemplos:
```
omite campos ‘Nome’;
omite campos ‘Nome’,’CPF’,’Salvar’;
```
### 2.3. Abre Modal

Abre a modal informada e executa as ações contidas no atributo “aoAbrir”. A modal deve ser referenciada pelo nome.
Ao iniciar uma funcionalidade, todas as modais devem estar ocultas. Portanto, uma modal só é aberta após esta ação ser executada.

Sintaxe:
```
<acao_abre_modal> ::= “abre modal ‘” <nome_modal> “’;”
```
Exemplos:
```
abre modal ‘Confirma Remoção do Registro’;
```
### 2.4. Fecha Modal

Fecha a modal informada. A modal deve ser referenciada pelo nome.

Sintaxe:
```
<acao_fecha_modal> ::= “fecha modal ‘” <nome_modal> “’;”
```
Exemplos:
```
fecha modal ‘Confirma Remoção do Registro’;
```
### 2.5. Exibe Mensagem

É comum os sistemas darem destaque às mensagens exibidas para o usuário. Estas mensagens geralmente utilizam um componente visual separado, no canto da tela ou em cor diferente. Por conta do destaque que esse conteúdo merece, a Konduto possui uma ação específica para ela.

Sintaxe:
```
<acao_exibe_mensagem> ::= “exibe mensagem ‘” <mensagem> “’;”
```
Exemplos:
```
exibe mensagem ‘Operação realizada com sucesso’;
```
### 2.6. Direciona

Direciona para outra página, a qual pode ser uma funcionalidade interna ou um link externo. A ação indica se a referência deve ser entendida como uma URL ou como o nome de uma funcionalidade do sistema. No segundo caso, o sistema deve deduzir a URL associada à funcionalidade referenciada.

Sintaxe:
```
<acao_direciona> ::= “direciona para ‘” <nome_funcionalidade> “’;”
                   | “direciona para url ‘” <url> “’;”
```
Exemplos:
```
direciona para ‘Cadastrar Pessoa’;
direciona para url ‘www.google.com’;
```
### 2.7. Comenta

Insere um comentário no código-fonte.

Sintaxe:
```
<acao_comenta> ::= “comenta ‘” <comentario_de_uma_linha> “’;”
```
Exemplos:
```
comenta ‘TODO verificar se esta eh a melhor alternativa’;
```
### 2.8. Declara Variável

Declara uma variável, especificando seu nome e seu tipo, mas não o seu valor.
O tipo pode ser primitivo (Texto, Número ou Booleano) ou composto.
Se o tipo for composto, deve ser o nome de uma entidade do sistema ou da funcionalidade.

Sintaxe:
```
<acao_declara_variavel> ::=
    “declara ‘” <nome_variavel> “’ do tipo ” <tipo_variavel> “;”
<tipo_variavel> ::= <tipo_primitivo> | <tipo_composto>
<tipo_primitivo> ::= “Texto” | “Número” | “Booleano”
<tipo_composto> ::= “‘” <nome_entidade> “’”
```
Exemplos:
```
declara ‘idade’ do tipo Número;
declara ‘pessoa’ do tipo ‘Pessoa’;
```
### 2.9. Atribui um Valor a um Dado

Declara uma variável, especificando seu nome e seu tipo, mas não o seu valor.
O tipo pode ser primitivo (Texto, Número ou Booleano) ou composto.
Se o tipo for composto, deve ser o nome de uma entidade do sistema ou da funcionalidade.

Sintaxe:
```
<acao_declara_variavel> ::=
    “declara ‘” <nome_variavel> “’ do tipo ” <tipo_variavel> “;”
<tipo_variavel> ::= <tipo_primitivo> | <tipo_composto>
<tipo_primitivo> ::= “Texto” | “Número” | “Booleano”
<tipo_composto> ::= “‘” <nome_entidade> “’”
```
Exemplos:
```
declara ‘idade’ do tipo Número;
declara ‘pessoa’ do tipo ‘Pessoa’;
```
