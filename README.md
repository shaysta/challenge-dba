
+A Educação - Desafio PostgreSQL
===================

[![N|Solid](https://maisaedu.com.br/hubfs/site-grupo-a/logo-mais-a-educacao.svg)](https://maisaedu.com.br/) 

O objetivo deste desafio é avaliar algumas competências técnicas consideradas fundamentais para candidatos ao cargo de DBA na Maior Plataforma de Educação do Brasil.

Será solicitado ao candidato que realize algumas tarefas baseadas em estrutura incompleta de tabelas relacionadas neste documento. Considere o PostgreSQL como SGDB ao aplicar conceitos e validações.

## Contexto

Você está lidando com um banco de dados multi-tenant que armazena informações acadêmicas sobre pessoas, instituições, cursos e matrículas de diferentes clientes (tenants). Sua tarefa será estruturar as tabelas, criar chaves primárias e estrangeiras, otimizar consultas, entre outras atividades. O desafio também envolve manipulação de dados em formato `jsonb`, com a necessidade implícita de construir índices apropriados.

## Estrutura das Tabelas

### 1. Tabela `tenant`
Representa diferentes clientes que utilizam o sistema. Pode ter cerca de 100 registros.

```sql
CREATE TABLE tenant (
    id SERIAL,
    name VARCHAR(100),
    description VARCHAR(255)
);
```
### 2.	Tabela `person` 
Contém informações sobre indivíduos. Esta tabela não está associada diretamente a um tenant. Estima-se ter 5.000.000 registros.

```sql
CREATE TABLE person (
    id SERIAL,
    name VARCHAR(100),
    birth_date DATE,
    metadata JSONB
);
```
### 3.	Tabela `institution`
Armazena detalhes sobre instituições associadas a diferentes tenants. Esta tabela terá aproximadamente 1.000 registros.

```sql
CREATE TABLE institution (
    id SERIAL,
    tenant_id INTEGER,
    name VARCHAR(100),
    location VARCHAR(100),
    details JSONB
);
```
### 4.	Tabela `course` 
Contém informações sobre cursos oferecidos por instituições, também associadas a um tenant. Deve ter cerca de 5.000 registros.

```sql
CREATE TABLE course (
    id SERIAL,
    tenant_id INTEGER,
    institution_id INTEGER,
    name VARCHAR(100),
    duration INTEGER,
    details JSONB
);
```

### 5.	Tabela `enrollment`
Armazena informações de matrículas, associadas a um tenant. Esta é a tabela com maior volume de dados, com cerca de 100.000.000 registros.

```sql
CREATE TABLE enrollment (
    id SERIAL,
    tenant_id INTEGER,
    institution_id INTEGER,
    person_id INTEGER,
    enrollment_date DATE,
    status VARCHAR(20)
);
```

## Tarefas

1. Identifique as chaves primárias e estrangeiras necessárias para garantir a integridade referencial. Defina-as corretamente. 
2. Construa índices que consideras essenciais para operações básicas do banco e de consultas possíveis para a estrutura sugerida.
3. Considere que em enollment só pode existir um único person_id por tenant e institution. Mas institution poderá ser nulo. Como garantir a integridade desta regra?
4. Caso eu queira incluir conceitos de exclusão lógica na tabela enrollment. Como eu poderia fazer? Quais as alterações necessárias nas definições anteriores?
5. Construa uma consulta que retorne o número de matrículas por curso em uma determinada instituição.Filtre por tenant_id e institution_id obrigatoriamente. Filtre também por uma busca qualquer -full search - no campo metadata da tabela person que contém informações adicionais no formato JSONB. Considere aqui também a exclusão lógica e exiba somente registros válidos.
6. Construa uma consulta que retorne os alunos de um curso em uma tenant e institution específicos. Esta é uma consulta para atender a requisição que tem por objetivo alimentar uma listagem de alunos em determinado curso. Tenha em mente que poderá retornar um número grande de registros por se tratar de um curso EAD. Use boas práticas. Considere aqui também a exclusão lógica e exiba somente registros válidos.
7. Suponha que decidimos particionar a tabela enrollment. Desenvolva esta ideia. Reescreva a definição da tabela por algum critério que julgues adequado. Faça todos os ajustes necessários e comente-os.
8. Sinta-se a vontade para sugerir e aplicar qualquer ajuste que achares relevante. Comente-os


## Critérios de avaliação
- Organização, clareza e lógica
- Utilização de boas práticas
- Documentação justificando o porquê das escolhas

## Instruções de entrega
1. Crie um fork do repositório no seu GitHub
2. Faça o push do código desenvolvido no seu Github
3. Informe ao recrutador quando concluir o desafio junto com o link do repositório
4. Após revisão do projeto, em conjunto com a equipe técnica, deixe seu repositório privado


Resposta 1, 2 e 3


CREATE TABLE tenant (
    id SERIAL,
    name VARCHAR(100),
    description VARCHAR(255)
CONSTRAINT pk_id_tenant PRIMARY KEY (id)
);

CREATE TABLE person (
    id SERIAL,
    name VARCHAR(100),
    birth_date DATE,
    metadata JSONB
CONSTRAINT pk_id_person PRIMARY KEY (id)
);

CREATE INDEX ON birth_date_idx person (birth_date);

CREATE TABLE institution (
    id SERIAL,
    tenant_id INTEGER,
    name VARCHAR(100),
    location VARCHAR(100),
    details JSONB
CONSTRAINT pk_id_institution PRIMARY KEY (id)
 FOREIGN KEY (tenant_id) REFERENCES tenant (id)
);

CREATE TABLE course (
    id SERIAL,
    tenant_id INTEGER,
    institution_id INTEGER,
    name VARCHAR(100),
    duration INTEGER,
    details JSONB
CONSTRAINT pk_id_course PRIMARY KEY (id)
FOREIGN KEY (tenant_id) REFERENCES tenant (id)
FOREIGN KEY (institution_id) REFERENCES institution (id)
);

CREATE INDEX ON institution_idx course (institution_id);


CREATE TABLE enrollment (
    id SERIAL,
    tenant_id INTEGER,
    institution_id INTEGER null,
    person_id INTEGER,
    enrollment_date DATE,
    status VARCHAR(20)
CONSTRAINT pk_id_enrollment PRIMARY KEY (id),
FOREIGN KEY (tenant_id) REFERENCES tenant (id),
FOREIGN KEY (institution_id) REFERENCES institution (id),
FOREIGN KEY (person_id) REFERENCES person (id));
CREATE INDEX ON enrollment_date_idx enrollment (enrolment_date);

Resposta 4

Exclusao logica
Soluçäo
Incluir uma coluna como flag, seria habilitada = 1 quando a coluna fosse alterada para excluida e por default ficaria 0.

CREATE TABLE enrollment (
    id SERIAL,
    tenant_id INTEGER,
    institution_id INTEGER null,
    person_id INTEGER,
    enrollment_date DATE,
    detected flag, 
    status VARCHAR(20)
CONSTRAINT pk_id_enrollment PRIMARY KEY (id),
FOREIGN KEY (tenant_id) REFERENCES tenant (id),
FOREIGN KEY (institution_id) REFERENCES institution (id),
FOREIGN KEY (person_id) REFERENCES person (id));
CREATE INDEX ON enrollment_date_idx enrollment (enrolment_date);

5 .Construa uma consulta que retorne o número de matrículas por curso em uma determinada instituição.Filtre por tenant_id e institution_id obrigatoriamente. Filtre também por uma busca qualquer -full search - no campo metadata da tabela person que contém informações adicionais no formato JSONB. Considere aqui também a exclusão lógica e exiba somente registros válidos.


SELECT count(*), c.id_course, a.institution_id, a.tenant_id FROM enrollment a
inner join institution b
on a.id_insitution = b.d_insitution
inner join course c
on on a.id_insitution = c.d_insitution
group by a.institution_id, a.tenant_id

SELECT info->>'name' as nome FROM person WHERE id = 1;
and deleted =0


6. Construa uma consulta que retorne os alunos de um curso em uma tenant e institution específicos. Esta é uma consulta para atender a requisição que tem por objetivo alimentar uma listagem de alunos em determinado curso. Tenha em mente que poderá retornar um número grande de registros por se tratar de um curso EAD. Use boas práticas. Considere aqui também a exclusão lógica e exiba somente registros válidos.

CREATE OR REPLACE FUNCTION select_curso (IN id_tenant VARCHAR, id_institution target VARCHAR) RETURNS VOID AS $$ BEGIN
 declare @id_tenant int, @id_institution int, 


SELECT c.id_course, a.institution_id, d.name, d.id FROM enrollment a
inner join institution b
on a.id_insitution = b.d_insitution
inner join course c
on on a.id_insitution = c.d_insitution
inner join person d
on a.id_tenant = b.tenant_id
where id_instittion=@id_institution and id_tenant = @id_tenant



7. Suponha que decidimos particionar a tabela enrollment. Desenvolva esta ideia. Reescreva a definição da tabela por algum critério que julgues adequado. Faça todos os ajustes necessários e comente-os.

CREATE TABLE enrollment (
    id SERIAL,
    tenant_id INTEGER,
    institution_id INTEGER null,
    person_id INTEGER,
    enrollment_date DATE,
    status VARCHAR(20)
CONSTRAINT pk_id_enrollment PRIMARY KEY (id),
FOREIGN KEY (tenant_id) REFERENCES tenant (id),
FOREIGN KEY (institution_id) REFERENCES institution (id),
FOREIGN KEY (person_id) REFERENCES person (id));
CREATE INDEX ON enrollment_date_idx enrollment (enrolment_date)
PARTITION BY RANGE (id);


CREATE TABLE enrollment1 PARTITION OF enrollment
    FOR VALUES FROM (1) TO (100000');

CREATE TABLE enrollment2 PARTITION OF enrollment
    FOR VALUES FROM (100000) TO (1000000');

CREATE TABLE enrollment2PARTITION OF enrollment
    FOR VALUES FROM (1000000) TO (100000000');

