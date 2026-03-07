# Primer Examen Parcial - Despliegue y Operación (DEVOPS)

**Nombre del estudiante:** JULIO DAVID ARNADO RIVERA

## Índice
- [Funcionamiento del CRUD](#funcionamiento-del-crud)
- [Ejecución del pipeline](#ejecución-del-pipeline)
- [Ejecución exitosa del pipeline](#ejecución-exitosa-del-pipeline)

## :books: Descripción

Este programa implementa un **CRUD (Create, Read, Update, Delete)** para gestionar productos utilizando **Python** y un **archivo JSON** como almacenamiento de datos.

## :books: Funcionamiento del CRUD

## 1. Carga y guardado de datos

### `_load_data()`

Esta función se encarga de leer los productos desde el archivo JSON.

**Funcionamiento:**

- Verifica si el archivo `products.json` existe.
- Si no existe, devuelve una lista vacía.
- Si existe, abre el archivo en modo lectura.
- Convierte el contenido JSON a una lista de Python usando `json.load()`.

```python
def _load_data():
```

### `_save_data(data)`

Esta función guarda los datos en el archivo JSON.

**Pasos:**

- Abre el archivo `products.json` en modo escritura.
- Convierte la lista de productos a formato JSON con `json.dump()`.
- Usa `indent=4` para que el archivo sea más legible.

```python
def _save_data(data):
```
---

## 2. Create (Crear producto)

### `create_product(product)`

Agrega un nuevo producto a la lista.

**Proceso:**

1. Carga los productos existentes.
2. Agrega el nuevo producto con `append`.
3. Guarda nuevamente todos los datos en el archivo.
4. Retorna el producto creado.

```python
def create_product(product):
```
---

## 3. Read (Leer productos)

### `read_products()`

Devuelve la lista completa de productos almacenados.

**Proceso:**

- Llama a `_load_data()`.
- Retorna la lista de productos.

```python
def read_products():
```
---

## 4. Update (Actualizar producto)

### `update_product(product_id, new_data)`

Permite modificar los datos de un producto usando su `id`.

**Proceso:**

1. Carga los productos existentes.
2. Recorre la lista buscando el producto con el `id` indicado.
3. Si lo encuentra, actualiza los campos usando `update()`.
4. Guarda los cambios en el archivo JSON.
5. Retorna el producto actualizado.

```python
def update_product(product_id, new_data):
```
Si no encuentra el producto, retorna:
```
None
```
---

## 5. Delete (Eliminar producto)

### `delete_product(product_id)`

Elimina un producto usando su `id`.

**Proceso:**

1. Carga los productos actuales.
2. Crea una nueva lista excluyendo el producto con ese `id`.
3. Si la longitud de la lista no cambió, significa que el producto no existía.
4. Si cambió, guarda la nueva lista en el archivo.

```python
def delete_product(product_id):
```

### Retorna

- `True` → si el producto fue eliminado.
- `False` → si el producto no existía.

---

## 6. Interfaz de consola

El bloque:

```python
if __name__ == "__main__":
```

Permite ejecutar el programa desde la terminal con un menú interactivo.

### Opciones disponibles

1. Crear producto
2. Ver productos
3. Actualizar producto
4. Eliminar producto
5. Salir

Cada opción llama a la función correspondiente del CRUD.

### Ejemplo de ejecución
![crud-products](https://github.com/user-attachments/assets/38ca8c1a-38c2-4ba6-83f2-6a51d5584a6b)


## :books: Ejecución del pipeline

Este pipeline de **GitHub Actions** automatiza el proceso de **integración continua (CI)** y **despliegue (CD)** del proyecto.

---

## 1. Eventos que activan el pipeline

El workflow se ejecuta cuando ocurre alguno de estos eventos:

- **Push** a la rama `main`
- **Pull Request** hacia la rama `main`

```yaml
on:
  push:
    branches: ["main"]
  pull_request:
    branches: ["main"]
```

Esto asegura que cada cambio importante en la rama principal sea **verificado automáticamente**.

---

## 2. Job: `build-test`

Este job se encarga de **preparar el entorno, analizar la calidad del código y ejecutar las pruebas**.

### Entorno

Se ejecuta en una máquina virtual con:

```yaml
runs-on: ubuntu-latest
```

### Pasos del job

#### 1. Clonar el repositorio

Descarga el código del repositorio en el runner.

```yaml
- name: Checkout repo
  uses: actions/checkout@v3
```

#### 2. Configurar Python

Instala **Python 3.10** en el entorno de ejecución.

```yaml
- name: Set up Python
  uses: actions/setup-python@v4
  with:
    python-version: "3.10"
```

#### 3. Instalar dependencias

Instala las librerías necesarias definidas en `requirements.txt`.

```yaml
- name: Install dependencies
  run: pip install -r requirements.txt
```

#### 4. Analizar el código (Lint)

Ejecuta **flake8** para detectar errores de estilo o problemas en el código.

```yaml
- name: Lint code
  run: flake8 .
```

#### 5. Ejecutar pruebas

Corre las pruebas automáticas con **pytest**.

```yaml
- name: Run tests
  run: PYTHONPATH=. pytest --maxfail=1 --disable-warnings -q
```

Opciones utilizadas:

- `PYTHONPATH=.` → Permite importar módulos del proyecto.
- `--maxfail=1` → Detiene la ejecución al primer error.
- `--disable-warnings` → Oculta advertencias.
- `-q` → Modo silencioso.

Si **alguno de estos pasos falla, el pipeline se detiene**.

---

## 3. Job: `docker-deploy`

Este job se encarga de **construir y ejecutar el contenedor Docker**.

Solo se ejecuta si el job anterior (`build-test`) termina correctamente.

```yaml
docker-deploy:
  runs-on: ubuntu-latest
  needs: build-test
```

### Pasos del job

#### 1. Clonar el repositorio

Descarga nuevamente el código para usarlo en la construcción de la imagen.

```yaml
- name: Checkout repo
  uses: actions/checkout@v3
```

#### 2. Construir la imagen Docker

Crea una imagen Docker llamada `crud-products`.

```bash
docker build -t crud-products .
```

Configuración en el pipeline:

```yaml
- name: Build Docker image
  run: docker build -t crud-products .
```

#### 3. Ejecutar el contenedor

Levanta el contenedor en segundo plano y expone el puerto `8000`.

```bash
docker run -d -p 8000:8000 crud-products
```

Configuración en el pipeline:

```yaml
- name: Run container
  run: docker run -d -p 8000:8000 crud-products
```

---

## :books: Ejecución exitosa del pipeline

### :computer: Pipeline ejecutado exitosamente
<img width="1906" height="432" alt="pipeline1" src="https://github.com/user-attachments/assets/53403e05-9356-4fc0-90e7-d0acf8dfa739" />

### :computer: Resumen del pipeline
<img width="1873" height="571" alt="pipeline2" src="https://github.com/user-attachments/assets/59c0b953-574d-4fdd-8e7c-9d5eb96c8e82" />

### :computer: Job build-test
<img width="1891" height="725" alt="pipeline3" src="https://github.com/user-attachments/assets/42e02336-d789-4faf-aa91-08510d0a1127" />

### :computer: Job docker-deploy
<img width="1900" height="707" alt="pipeline4" src="https://github.com/user-attachments/assets/c284b3a8-6d3e-4397-b272-a73e45251cd4" />

