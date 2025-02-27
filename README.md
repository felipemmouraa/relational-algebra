# Exemplo de Modelagem de Dados e Consultas SQL com Dados Simulados

Este documento apresenta um cenário de modelagem de dados que contempla uma relação 1:N e uma relação N:N, utilizando as tabelas **Professores**, **Cursos**, **Alunos** e a tabela associativa **Matriculas**.  
A seguir, são demonstrados alguns comandos de inserção (simulados) para popular as tabelas, as consultas SQL propostas (incluindo uma consulta com junção de três entidades), e também consultas simples de seleção e projeção.

---

## Estrutura e Inserção de Dados

### Tabela: Professores  
- **Atributos:**  
  - `ProfessorID` (PK)  
  - `Nome`  
  - `Titulacao`  

```sql
INSERT INTO Professores (ProfessorID, Nome, Titulacao) VALUES
(1, 'Dr. Silva', 'Adjunto'),
(2, 'Dra. Souza', 'Titular');
```

### Tabela: Cursos  
- **Atributos:**  
  - `CursoID` (PK)  
  - `Nome`  
  - `Categoria`  
  - `Valor` (valor contínuo, ex.: custo do curso)  
  - `ProfessorID` (FK referenciando Professores)  
- **Relação 1:N:** Cada professor pode ministrar vários cursos, mas cada curso possui apenas um professor.

```sql
INSERT INTO Cursos (CursoID, Nome, Categoria, Valor, ProfessorID) VALUES
(1, 'Introdução à Tecnologia', 'Tecnologia', 1500, 1),
(2, 'História da Arte', 'Humanas', 1200, 2),
(3, 'Programação Avançada', 'Tecnologia', 1800, 1);
```

### Tabela: Alunos  
- **Atributos:**  
  - `AlunoID` (PK)  
  - `Nome`  
  - `DataNascimento`  

```sql
INSERT INTO Alunos (AlunoID, Nome, DataNascimento) VALUES
(1, 'Ana', '2000-05-10'),
(2, 'Bruno', '1999-08-15'),
(3, 'Carlos', '2001-12-20');
```

### Tabela: Matriculas  
- **Atributos:**  
  - `MatriculaID` (PK)  
  - `AlunoID` (FK referenciando Alunos)  
  - `CursoID` (FK referenciando Cursos)  
  - `DataMatricula`  
- **Relação N:N:** Representa a associação entre Alunos e Cursos.

```sql
INSERT INTO Matriculas (MatriculaID, AlunoID, CursoID, DataMatricula) VALUES
(1, 1, 1, '2024-01-15'),
(2, 2, 1, '2024-01-16'),
(3, 3, 3, '2024-01-17'),
(4, 2, 3, '2024-01-18'),
(5, 1, 2, '2024-01-19'); -- Curso de 'Humanas'
```

---

## Consulta Complexa (Junção de Três Entidades)

**Enunciado da Consulta:**  
Liste, para cada professor com titulação `'Adjunto'`, a quantidade de alunos matriculados em seus cursos, considerando apenas os cursos da categoria `'Tecnologia'` cujo valor esteja entre 1000 e 2000.

**SQL:**

```sql
SELECT COUNT(m.AlunoID) AS TotalAlunos, p.Nome AS NomeProfessor
FROM Professores p
JOIN Cursos c ON p.ProfessorID = c.ProfessorID
JOIN Matriculas m ON c.CursoID = m.CursoID
WHERE p.Titulacao = 'Adjunto'
  AND c.Categoria = 'Tecnologia'
  AND c.Valor BETWEEN 1000 AND 2000
GROUP BY p.Nome;
```

**Resultado:**  
- Professor **Dr. Silva** (ID 1, titulação 'Adjunto') ministra dois cursos:
  - Curso 1: 'Introdução à Tecnologia' (Categoria: Tecnologia, Valor: 1500)  
    → Matrículas: MatriculaID 1 (Aluno Ana) e 2 (Aluno Bruno)
  - Curso 3: 'Programação Avançada' (Categoria: Tecnologia, Valor: 1800)  
    → Matrículas: MatriculaID 3 (Aluno Carlos) e 4 (Aluno Bruno)
- O curso com ID 2 pertence a 'Humanas' e não é considerado.

Portanto, o resultado da consulta será:

| NomeProfessor | TotalAlunos |
|---------------|-------------|
| Dr. Silva     | 4           |

---

## Conversão para Álgebra Relacional

A consulta SQL acima pode ser convertida na seguinte equação em álgebra relacional, com aplicação dos filtros o mais cedo possível (push-down):

```plaintext
γ_{p.Nome; COUNT(m.AlunoID) → TotalAlunos} (
  (σ_{p.Titulacao = 'Adjunto'}(Professores)
    ⨝_{Professores.ProfessorID = Cursos.ProfessorID} 
      σ_{c.Categoria = 'Tecnologia' ∧ c.Valor ≥ 1000 ∧ c.Valor ≤ 2000}(Cursos)
  )
  ⨝_{Cursos.CursoID = Matriculas.CursoID} Matriculas
)
```

---

## Consultas Simples

### 1. Consulta de Seleção  
Seleciona todos os dados do aluno chamado 'Ana'.

```plaintext
σ_{Nome = 'Ana'}(Alunos)
```

**Resultado Simulado:**

| AlunoID | Nome | DataNascimento |
|---------|------|----------------|
| 1       | Ana  | 2000-05-10     |

### 2. Consulta de Projeção  
Seleciona apenas os atributos `Nome` e `DataNascimento` de todos os alunos.

```plaintext
π_{Nome, DataNascimento}(Alunos)

```

**Resultado Simulado:**

| Nome   | DataNascimento |
|--------|----------------|
| Ana    | 2000-05-10     |
| Bruno  | 1999-08-15     |
| Carlos | 2001-12-20     |
