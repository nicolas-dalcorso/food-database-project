# Universo do Discurso
## $\S 1$ Introdução
O presente trabalho busca modelar um aplicativo de interação entre empresas do ramo alimentício e a população no geral. Os conhecimentos construídos a partir de disciplina de Fundamentos de Bancos de Dados, portanto, estão sendo utilizados na modelagem de estruturas (entidades e relacionamentos) que visam representar clientes, empresas e alimentos (as "grandes classes" do trabalho), bem como outras relações, descritas abaixo.

As relações estabelecidas buscam 
1. permitir que clientes encontrem informações acerca de empresas de alimentação e de alimentos consumidos;
2. permitir que empresas encontrem informações sobre seus possíveis produtos

As entidades e relações descritas neste documento fazem uso de [database markup language](https://dbml.dbdiagram.io/docs/) para expressar as estruturas do banco de dados de forma mais simples na representação dos diagramas ER, e fazem uso da sintaxe do PostgreSQL para demonstrar uma primeira versão, possivelmente passível de modificações, das tabelas a serem implementadas. Os relacionamentos ER foram criados através da aplicação de database markup language sobre o web-aplicativo [dbdiagram](https://dbdiagram.io/d).

---
## $\S 2$ Para que serve este aplicativo?
Tanto para empresas no geral quanto para usuários, o aplicativo fornece informações sobre
- valores nutricionais de diversos alimentos;
- uma possível precificação de alimentos individuais ou de refeições;
- a receita de uma refeição; e
- formas de categorizar e encontrar alimentos, refeições ou restaurantes, de acordo com suas preferências.

---
## $\S 3$ Organização, entidades e relações
### $\S 3.1$ `Food`
A principal entidade de todo o sistema refere-se aos objetos `Food`, que marcam alimentos individuais. Objetos desta categoria são bem descritas através da tabela abaixo:

```sql
TABLE Foods(
	id integer NOT NULL,
	name varchar NOT NULL,
	price float,
	primary key (food_id)	
)
```

O atributo `price` é introduzido em um segundo momento, a partir do acesso à outra tabela.

### $\S 3.2$ `FoodData`
A entidade `FoodData` relaciona-se diretamente à entidade `Food` como um descritor dos **valores nutricionais** de um elemento `Food`. Seus valores foram extraídos do [dataset 'Food, Vitamins, Minerals, Macronutrient', encontrado em Kaggle.com](https://www.kaggle.com/datasets/mexwell/food-vitamins-minerals-macronutrient/). Um objeto `FoodData` será representado por

```SQL
TABLE FoodData(
	food_id integer NOT NULL,
	description varchar NOT NULL,
	category varchar NOT NULL,

	-- diversos valores, referentes aos nutrientes

	primary key (category, description)
	foreign key (food_id) references Food
)
```

> [!tldr] Valores nutricionais associados à `FoodData`
> Os diversos valores, indicados pelo comentário acima, são 
> 
> `"Nutrient Data Bank Number", "Data.Alpha Carotene", "Data.Beta Carotene","Data.Beta Cryptoxanthin", "Data.Carbohydrate", "Data.Cholesterol", "Data.Choline", "Data.Fiber", "Data.Lutein and Zeaxanthin", "Data.Lycopene", "Data.Niacin", "Data.Protein", "Data.Retinol", "Data.Riboflavin", "Data.Selenium", "Data.Sugar Total", "Data.Thiamin", "Data.Water", "Data.Fat.Monosaturated Fat", "Data.Fat.Polysaturated Fat", "Data.Fat.Saturated Fat", "Data.Fat.Total Lipid", "Data.Major Minerals.Calcium", "Data.Major Minerals.Copper", "Data.Major Minerals.Iron", "Data.Major Minerals.Magnesium", "Data.Major Minerals.Phosphorus", "Data.Major Minerals.Potassium", "Data.Major Minerals.Sodium", "Data.Major Minerals.Zinc", "Data.Vitamins.Vitamin A - RAE", "Data.Vitamins.Vitamin B12", "Data.Vitamins.Vitamin B6", "Data.Vitamins.Vitamin C", "Data.Vitamins.Vitamin E", "Data.Vitamins.Vitamin K"`

conforme encontrados no *dataset* descrito.

---
### $\S 3.2$ `Recipe`, `RecipeTimings`
Uma receita, ou `Recipe`, relaciona-se diretamente com as duas entidades descritas anteriormente. Para definir uma receita, é necessário que tenhamos uma lista de ingredientes; cada um destes ingredientes refere-se a um elemento `Food`; por sua vez, cada um desses ingredientes estará referenciado por um `FoodData` que contém seus valores nutricionais. Uma vez precificados cada um dos ingredientes, a receita como um todo pode ser precificada.

```sql
Table Recipes(
	title VARCHAR NOT NULL,
	category VARCHAR NOT NULL,
	rt_id INTEGER NOT NULL,
	ingridients INTEGER[] NOT NULL,
	servings INTEGER,
	yield VARCHAR,
	price FLOAT,
	primary key (title),
	foreign key (ingridients) REFERENCES id (Food)
	foreign key (rt_id) REFERENCES rt_id (RecipeTimes),
)
```

onde o atributo `Recipe.ingridients` é uma lista de inteiros no qual cada valor refere-se a um `Food.id` único, representativo de um ingrediente da receita. Além disto, `Recipe.rt_id` refere-se ao identificador de uma entrada na tabela de tempos de preparo das receitas:

```sql
Table RecipeTimings(
	recipe_title VARCHAR NOT NULL,
	rt_id INTEGER NOT NULL,
	prep_time VARCHAR NOT NULL,
	cook_time VARCHAR NOT NULL,
	total_time VARCHAR,
	primary key (rt_id),
	foreign key (recipe_title) REFERENCES (Recipe)
)
```

Podemos definir estas tabelas utilizando database markup language, o que nos garante uma organização do tipo:

```database-markup-language
-- Definição das entidades
Table Foods{
	id integer [primary key]
	name varchar
	price float
}  

-- A fim de econimizar espaço, apenas quatro nutrientes são apresentados
Table FoodData{
	food_id integer
	description varchar [primary key]
	category varchar [primary key]
	alpha_carotene float
	beta_carotene float
	cryptoxanthin float
	carbohydrate float
}

Table Recipes{
	title varchar [primary key]
	category varchar
	rt_id integer
	ingridients integer[]
	servings integer
	yield varchar
	price float
}

  

Table RecipeTimings{
	recipe_title varchar
	rt_id integer [primary key]
	prep_time float
	cook_time float
	total_time float
}
  
-- Definição das relações
Ref: FoodData.food_id < Foods.id
Ref: Recipes.ingridients <> Foods.id
Ref: RecipeTimings.rt_id < Recipes.rt_id
Ref: RecipeTimings.recipe_title < Recipes.title
```

![[Pasted image 20231203203108.png]]

---
### $\S 3.3$ Usuários, clientes, empresas e restaurantes
Uma entidade `User` é um tipo geral sobre o qual todas as demais entidades aqui descritas pertencem.

```SQL
Table Users(
	id INTEGER NOT NULL,
	type VARCHAR NOT NULL,
	primary key (id),
	check (type in ('Client', 'Restaurant'))
)
```

```sql
Table Clients(
	id INTEGER NOT NULL,
	name VARCHAR NOT NULL,

	-- relações e atributos a serem implementados

	primary key (name),
	foreign key (id) REFERENCES (Users)
)
```

```sql
Table Restaurants(
	id INTEGER NOT NULL,
	c_id INTEGER NOT NULL,
	name VARCHAR NOT NULL,
	category VARCHAR NOT NULL,

	-- relações e atributos a serem implementados

	primary key (name),
	foreign key (c_id) REFERENCES  c_id (Companies),
	foreign key (category) REFERENCES category (Foods) 
	foreign key (id) REFERENCES id (Users)
)
```

```sql
Table Companies(
	c_id INTEGER NOT NULL,
	name VARCHAR NOT NULL,

	-- relações e atributos a serem implementados

	primary key (company_id)
)
```

Dessa forma, cada `Restaurant` é associado a uma `Company`. Uma empresa, naturalmente, está associada a nenhum, um ou mais restaurantes. Cada restaurante, por sua vez, pertencerá a uma categoria, como "culinária japonesa" ou qualquer outra das categorias que o atributo `Recipe.category` pode receber. É evidente, no entanto, que é possível um restaurante pode servir refeições de mais de uma categoria, mas esta especialização será utilizada na implementação do aplicativo final.

O acesso aos dados sempre se dará mediante ao acesso de um `User`. As restrições e *features* do produto final, entretanto, discriminarão entre um usuário que representa um cliente e entre um usuário que representa uma empresa ou restaurante.

Em database markup language, estas estruturas foram implementadas até o momento sob a seguinte forma:

```database-markup-language
Table Users{
	id integer [primary key]
	type varchar|
}

Table Clients{
	name varchar [primary key]
	id integer
}

Table Restaurants{
	name varchar [primary key]
	id integer
	c_id integer
	category varchar
}  

Table Companies{
	c_id integer [primary key]
	name varchar
}
 
Ref: Restaurants.c_id > Companies.c_id
Ref: Restaurants.id < Users.id
Ref: Restaurants.category <> FoodData.category
Ref: Clients.id > Users.id
```

![[Pasted image 20231203204331.png]]

---

### $\S 3.4$ Entidades e relacionamentos direcionados à implementação
As entidades `Tag`, `Price` e ... são *internas* à implementação do programa-final, e tem atuação na coleta, processamento ou exposição dos dados.

```SQL
Table Prices(
	food_id integer
	food_name varchar
	price float
	primary key (food_id, food_name)
	foreign key (food_id) REFERENCES id (Foods)
)
```

>[!abstract] Construção dos preços
>A implementação da *feature* de precificação dos alimentos (objetos `Food`) se dará através de um cálculo sobre todas as entradas na tabela `Prices` nas quais o objeto `Food` é referenciado.
>Se, por exemplo, a precificação for realizada através do produto de uma média de $n$ objetos `Price` que referenciam um determinado objeto `Food`, por um fator de lucro $k$, diga-se
>$$\text{Price}_{\text{Food, Final}} = k \cdot \frac{1}{n} \cdot \sum_{i=0}^{n} \text{~Price}_{i}$$
>onde cada $\text{Price}_{i}$ será extraído da tabela `Prices` para formar o $\text{Price}_{\text{Food, Final}}$.

```sql
Table Descriptors(
	name varchar NOT NULL,
	object varchar NOT NULL,
	description varchar NOT NULL,
	primary key (name)
)
```

> [!abstract] Descritores de atributos para `FoodData`
> A tabela `Descriptors` reúne todos os descritores necessários para descrever os atributos nutricionais dos alimentos. Assim, uma vez implementado a aplicação final, será possível receber a descrição de um atributo, `carbohydrate`, por exemplo, através de
>  
> `>  describe carbohydrate`
> 
> o que retornará a projeção do atributos `(name, description)` na tabela `Descriptors` onde `Descriptors.name = 'carbohydrate'`.

```sql
Table Queries(
	id INTEGER NOT NULL,
	query VARCHAR NOT NULL,
	result VARCHAR
	primary key (id)
)
```

> [!abstract] Armazenar os resultados de pesquisas feitas por usuários
> As pesquisas feitas por usuários poderão ser armazenadas em uma tabela, `Queries`, composta por um identificador único, a *query* do usuário e o resultado obtido pelo sistema. Desta forma, tanto uma organização interna voltada para melhoramentos (análise, por exemplo, dos tipos de pesquisas mais frequentemente feitas), quanto a análise para descoberta de possíveis *bugs* ou *queries* que retornam resultados indesejados ou errôneos.
