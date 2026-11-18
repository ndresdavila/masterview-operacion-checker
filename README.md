# Masterview — operación documental

Mapa de un sistema de documentos para **Masterview S.A.**: un formulario arma un Word, puede enviarlo por correo, lo entrega a un dashboard y un checker compara los documentos entre sí.

Este repositorio no duplica el código. Cada pieza vive en su propio servicio. Aquí está el recorrido y el enlace a cada uno.

## Recorrido

1. La persona completa el formulario y obtiene el Word, en el navegador o pidiéndoselo al generador.
2. Si el envío es por correo, el generador llama al servicio de correo y adjunta el documento.
3. El formulario entrega el mismo documento al dashboard.
4. El dashboard guarda el pendiente y archiva el archivo en Google Drive.
5. El personal completa los datos que faltan. El servicio genera el documento final y lo archiva.
6. El checker queda fuera de esa cadena. Compara documentos relacionados y marca diferencias de transcripción.

```mermaid
flowchart LR
  cliente["Usuario"] --> form["Formulario"]
  form --> gen["Generador de Word"]
  gen --> mail["Correo"]
  form --> hbl["Dashboard"]
  hbl --> redis["Cola de pendientes"]
  hbl --> drive["Google Drive"]
  staff["Personal"] --> hbl
  checker["Checker"] --> informe["Diferencias"]
```

## Secuencia

```mermaid
sequenceDiagram
  actor Usuario
  participant Form as Formulario
  participant Gen as Generador
  participant Mail as Correo
  participant Dash as Dashboard
  participant Drive as Google Drive
  actor Staff as Personal

  Usuario->>Form: Completa el formulario
  alt Word en el navegador
    Form->>Form: Rellena la plantilla
  else Word en el servidor
    Form->>Gen: Pide el documento
    Gen-->>Form: archivo .docx
  end
  opt Enviar por correo
    Form->>Gen: Pide generar y enviar
    Gen->>Mail: documento y destinatarios
    Mail-->>Usuario: correo con el Word
  end
  Form->>Dash: Entrega el documento
  Dash->>Dash: Guarda el pendiente
  Dash->>Drive: Archiva el archivo
  Staff->>Dash: Completa los datos que faltan
  Dash->>Drive: Archiva el documento final
```

## Checker

El checker no participa en el envío. Lee Word, PDF y Excel, normaliza texto y números, y devuelve una lista de diferencias.

```mermaid
flowchart LR
  docs["Documentos relacionados"] --> checker["Checker"]
  checker --> out["Lista de diferencias"]
```

## Repositorios

| Repositorio | Qué hace | Acceso |
|---|---|---|
| [masterview-formulario-proforma](https://github.com/ndresdavila/masterview-formulario-proforma) | Formulario en React. Arma el Word, pide el correo y entrega el documento al dashboard. | Público |
| [masterview-generador-documento](https://github.com/ndresdavila/masterview-generador-documento) | API en Python. Recibe JSON y devuelve el `.docx`. Si hace falta, llama al correo. | Público |
| [masterview-enviador-correo](https://github.com/ndresdavila/masterview-enviador-correo) | API en Python. Adjunta el documento y lo envía por correo. | Público |
| hbl-dashboard | API en Node y dashboard en React. Cola de pendientes, Google Drive y documento final. | Privado |
| [masterview-document-checker](https://github.com/ndresdavila/masterview-document-checker) | Compara documentos y marca diferencias. | Público |

El dashboard queda privado. Los otros cuatro repositorios se pueden abrir desde aquí.

La lectura de los campos está hecha con plantillas y parsers. Esa es la capa donde se conecta un modelo como Claude o Gemini cuando el documento no cabe en una plantilla fija.

## Cómo lo opero

Shippo es un agente de Cursor, con Grok, en la misma cuenta con la que mantengo estos repos. Revisa si hubo cambios en los documentos. Si hace falta tocar código, lo propone primero. Después de la confirmación, corre las pruebas y publica la versión nueva.

```mermaid
flowchart LR
  cambios["Cambios en documentos"] --> shippo["Shippo"]
  shippo --> aviso["Aviso"]
  aviso --> ok["Confirmación"]
  ok --> tests["Pruebas"]
  tests --> pub["Versión publicada"]
```

Shippo no es un servicio de este sistema. El historial de esas corridas está en Cursor.
