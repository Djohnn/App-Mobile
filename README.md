# Guia de Uso - Aplicativo Flet

## Instalação de Dependências e criação do ambiente virtual

Antes de iniciar o desenvolvimento do aplicativo crie o ambiente virtual,

```sh
python3 -m venv venv
source venv/bin/activate
```

Instale as dependências necessárias:

```sh
pip install flet
pip install requests
```

## Estrutura Mínima do Aplicativo Flet

Crie um arquivo Python e adicione o seguinte código básico para iniciar o aplicativo:

```python
import flet as ft
import requests

# URL base da API – ajuste conforme necessário
API_BASE_URL = "http://localhost:8000/api/treino"

def main(page: ft.Page):
    page.title = "Exemplo"
    page.vertical_alignment = ft.MainAxisAlignment.CENTER
    page.add()

if __name__ == "__main__":
    ft.app(target=main)
```

## Criando um Novo Aluno

### Campos de Entrada

```python
nome_field = ft.TextField(label="Nome")
email_field = ft.TextField(label="Email")
faixa_field = ft.TextField(label="Faixa")
data_nascimento_field = ft.TextField(label="Data de Nascimento (YYYY-MM-DD)")
create_result = ft.Text()

create_button = ft.ElevatedButton(text="Criar Aluno", on_click=criar_aluno_click)
```

### Função para Criar um Aluno

```python
def criar_aluno_click(e):
    payload = {
        "nome": nome_field.value,
        "email": email_field.value,
        "faixa": faixa_field.value,
        "data_nascimento": data_nascimento_field.value,
    }
    try:
        response = requests.post(API_BASE_URL + "/", json=payload)
        if response.status_code == 200:
            aluno = response.json()
            create_result.value = f"Aluno criado: {aluno}"
        else:
            create_result.value = f"Erro: {response.text}"
    except Exception as ex:
        create_result.value = f"Exceção: {ex}"
    page.update()
```

### Adicionando os Elementos na Página

```python
criar_aluno_tab = ft.Column([
    nome_field,
    email_field,
    faixa_field,
    data_nascimento_field,
    create_button,
    create_result,
], scroll=True)
```

## Criando Abas de Navegação

```python
tabs = ft.Tabs(
    selected_index=0,
    tabs=[
        ft.Tab(text="Criar Aluno", content=criar_aluno_tab),
    ]
)

page.add(tabs)
```

## Listando Todos os Alunos

### Criando a Tabela

```python
students_table = ft.DataTable(
    columns=[
        ft.DataColumn(ft.Text("ID")),
        ft.DataColumn(ft.Text("Nome")),
        ft.DataColumn(ft.Text("Email")),
        ft.DataColumn(ft.Text("Faixa")),
        ft.DataColumn(ft.Text("Data Nascimento")),
    ],
    rows=[],
)
list_result = ft.Text()
```

### Função para Listar os Alunos

```python
def listar_alunos_click(e):
    try:
        response = requests.get(API_BASE_URL + "/alunos/")
        if response.status_code == 200:
            alunos = response.json()
            students_table.rows.clear()
            for aluno in alunos:
                row = ft.DataRow(
                    cells=[
                        ft.DataCell(ft.Text(str(aluno.get("id", "")))),
                        ft.DataCell(ft.Text(aluno.get("nome", ""))),
                        ft.DataCell(ft.Text(aluno.get("email", ""))),
                        ft.DataCell(ft.Text(aluno.get("faixa", ""))),
                        ft.DataCell(ft.Text(aluno.get("data_nascimento", ""))),
                    ]
                )
                students_table.rows.append(row)
            list_result.value = f"{len(alunos)} alunos encontrados."
        else:
            list_result.value = f"Erro: {response.text}"
    except Exception as ex:
        list_result.value = f"Exceção: {ex}"
    page.update()
```

### Adicionando a Função à Interface

```python
list_button = ft.ElevatedButton(text="Listar Alunos", on_click=listar_alunos_click)
listar_alunos_tab = ft.Column([list_button, students_table, list_result], scroll=True)

ft.Tab(text="Listar Alunos", content=listar_alunos_tab)
```

## Marcar Aula Realizada

```python
email_aula_field = ft.TextField(label="Email do Aluno")
qtd_field = ft.TextField(label="Quantidade de Aulas", value="1")
aula_result = ft.Text()

def marcar_aula_click(e):
    try:
        qtd = int(qtd_field.value)
        payload = {"qtd": qtd, "email_aluno": email_aula_field.value}
        response = requests.post(API_BASE_URL + "/aula_realizada/", json=payload)
        if response.status_code == 200:
            mensagem = response.json()
            aula_result.value = f"Sucesso: {mensagem}"
        else:
            aula_result.value = f"Erro: {response.text}"
    except Exception as ex:
        aula_result.value = f"Exceção: {ex}"
    page.update()

aula_button = ft.ElevatedButton(text="Marcar Aula Realizada", on_click=marcar_aula_click)
aula_tab = ft.Column([email_aula_field, qtd_field, aula_button, aula_result], scroll=True)
```

## Consultar Progresso do Aluno

```python
email_progress_field = ft.TextField(label="Email do Aluno")
progress_result = ft.Text()

def consultar_progresso_click(e):
    try:
        email = email_progress_field.value
        response = requests.get(API_BASE_URL + "/student_progress/", params={"student_email": email})
        if response.status_code == 200:
            progress = response.json()
            progress_result.value = (
                f"Nome: {progress.get('student_name', '')}\n"
                f"Email: {progress.get('student_email', '')}\n"
                f"Faixa Atual: {progress.get('current_belt', '')}\n"
                f"Aulas Totais: {progress.get('total_lessons', 0)}\n"
                f"Aulas necessárias para a próxima faixa: {progress.get('lessons_needed_for_next_belt', 0)}"
            )
        else:
            progress_result.value = f"Erro: {response.text}"
    except Exception as ex:
        progress_result.value = f"Exceção: {ex}"
    page.update()

progress_button = ft.ElevatedButton(text="Consultar Progresso", on_click=consultar_progresso_click)
progresso_tab = ft.Column([email_progress_field, progress_button, progress_result], scroll=True)
```

## Atualizar Dados do Aluno

```python
    id_aluno_field = ft.TextField(label="ID do Aluno")
    nome_update_field = ft.TextField(label="Novo Nome")
    email_update_field = ft.TextField(label="Novo Email")
    faixa_update_field = ft.TextField(label="Nova Faixa")
    data_nascimento_update_field = ft.TextField(label="Nova Data de Nascimento (YYYY-MM-DD)")
    update_result = ft.Text()

    def atualizar_aluno_click(e):
        aluno_id = id_aluno_field.value
        if not aluno_id:
            update_result.value = "Por favor, insira o ID do aluno."
        else:
            payload = {
                "nome": nome_update_field.value,
                "email": email_update_field.value,
                "faixa": faixa_update_field.value,
                "data_nascimento": data_nascimento_update_field.value,
            }
            response = requests.put(API_BASE_URL + f"/alunos/{aluno_id}", json=payload)
            if response.status_code == 200:
                aluno = response.json()
                update_result.value = f"Aluno atualizado com sucesso: {aluno}"
            else:
                if response.json()['detail'][0]['type'] == 'date_from_datetime_parsing':
                    update_result.value = "Por favor insira a data de nascimento no formato YYYY-MM-DD"
                else:
                    update_result.value = f"Erro ao atualizar aluno: {response.text}"
                
        page.update()     
update_button = ft.ElevatedButton(text="Atualizar Aluno", on_click=atualizar_aluno_click)
atualizar_tab = ft.Column([
    id_aluno_field,
    nome_update_field,
    email_update_field,
    faixa_update_field,
    data_nascimento_update_field,
    update_button,
    update_result,
], scroll=True)
```

## Rodando o Projeto

Após finalizar as configurações, execute o seguinte comando para iniciar o aplicativo:

```sh
python nome_do_arquivo.py
```
Substitua nome_do_arquivo.py pelo nome do arquivo onde está o código do aplicativo. Isso iniciará o servidor e abrirá a interface gráfica do Flet no navegador.


