---
title: "fastapi openapi ui 离线方案"
description: "fastapi openapi ui 离线方案, 支持 swagger ui, scalar ui"
publishDate: "2025-07-07"
tags: ["fastapi", "openapi", "swagger", "scalar"]
---

# fastapi openapi ui 离线方案
## 1. swagger ui

```python title="swagger.py" showLineNumbers
from fastapi import FastAPI
from fastapi.openapi.docs import (
    get_swagger_ui_html, get_swagger_ui_oauth2_redirect_html
)
from starlette.requests import Request


def swagger(
        app: FastAPI,
        **kwargs
):
    _openapi_url = kwargs.pop("openapi_url", app.openapi_url)
    title = kwargs.pop("title", app.title)
    init_oauth = kwargs.pop("init_oauth", app.swagger_ui_init_oauth)
    _swagger_js_url = kwargs.pop("swagger_js_url", "https://cdn.jsdelivr.net/npm/swagger-ui-dist@5/swagger-ui-bundle.js")
    _swagger_css_url = kwargs.pop("swagger_css_url", "https://cdn.jsdelivr.net/npm/swagger-ui-dist@5/swagger-ui.css")

    _oauth2_redirect_url = "/swagger/oauth2-redirect.html"

    @app.get(_oauth2_redirect_url, include_in_schema=False)
    async def swagger_ui_oauth2_redirect_html():
        return get_swagger_ui_oauth2_redirect_html()

    @app.get("/swagger", include_in_schema=False)
    async def swagger_html(request: Request):
        root_path = request.scope.get("root_path", "").rstrip("/")
        openapi_url = root_path + _openapi_url
        swagger_js_url = root_path + _swagger_js_url
        swagger_css_url = root_path + _swagger_css_url

        oauth2_redirect_url = root_path + _oauth2_redirect_url
        return get_swagger_ui_html(
            openapi_url=openapi_url,
            title=title,
            oauth2_redirect_url=oauth2_redirect_url,
            init_oauth=init_oauth,
            swagger_js_url=swagger_js_url,
            swagger_css_url=swagger_css_url,
            **kwargs
        )


```

## 2. scalar ui

```python title="scalar.py" showLineNumbers
import json

from fastapi import FastAPI
from scalar_fastapi.scalar_fastapi import SearchHotKey, Layout, scalar_theme
from starlette.requests import Request
from starlette.responses import HTMLResponse


def get_scalar_ui_html(
        *,
        openapi_url: str,
        title: str,
        scalar_js_url: str = "https://cdn.jsdelivr.net/npm/@scalar/api-reference",
        scalar_proxy_url: str = "",
        scalar_favicon_url: str = "https://fastapi.tiangolo.com/img/favicon.png",
        scalar_theme: str = scalar_theme,
        layout: Layout = Layout.MODERN,
        show_sidebar: bool = True,
        hide_download_button: bool = False,
        hide_models: bool = False,
        dark_mode: bool = True,
        search_hot_key: SearchHotKey = SearchHotKey.K,
        hidden_clients: bool | dict[str, bool | list[str]] | list[str] = [],
        servers: list[dict[str, str]] = [],
        default_open_all_tags: bool = False,
        init_oauth: dict[str, str] = None,
) -> HTMLResponse:
    authentication = ""
    if init_oauth:
        authentication = f"""
        authentication: {json.dumps({
            "oAuth2": {
                "clientId": init_oauth.get("clientId"),
                "clientSecret": init_oauth.get("clientSecret"),
                "scopes": init_oauth.get("scopes").split(" "),
            }
        })}"""

    html = f"""
    <!DOCTYPE html>
    <html>
    <head>
    <title>{title}</title>
    <!-- needed for adaptive design -->
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1">
    <link rel="shortcut icon" href="{scalar_favicon_url}">
    <style>
      body {{
        margin: 0;
        padding: 0;
      }}
    </style>
    <style>
    {scalar_theme}
    </style>
    </head>
    <body>
    <noscript>
        Scalar requires Javascript to function. Please enable it to browse the documentation.
    </noscript>
    <script
      id="api-reference"
      data-url="{openapi_url}"
      data-proxy-url="{scalar_proxy_url}"></script>
    <script>
      var configuration = {{
        layout: "{layout.value}",
        showSidebar: {json.dumps(show_sidebar)},
        hideDownloadButton: {json.dumps(hide_download_button)},
        hideModels: {json.dumps(hide_models)},
        darkMode: {json.dumps(dark_mode)},
        searchHotKey: "{search_hot_key.value}",
        hiddenClients: {json.dumps(hidden_clients)},
        servers: {json.dumps(servers)},
        defaultOpenAllTags: {json.dumps(default_open_all_tags)},
        {authentication}
      }}

      document.getElementById('api-reference').dataset.configuration =
        JSON.stringify(configuration)
    </script>
    <script src="{scalar_js_url}"></script>
    </body>
    </html>
    """
    return HTMLResponse(html)


def scalar(
        app: FastAPI,
        **kwargs
):
    _openapi_url = kwargs.pop("openapi_url", app.openapi_url)
    _title = kwargs.pop("title", app.title)
    _servers = kwargs.pop("servers", app.servers)
    _init_oauth = kwargs.pop("init_oauth", app.swagger_ui_init_oauth)
    _scalar_js_url = kwargs.pop("scalar_js_url", "https://cdn.jsdelivr.net/npm/@scalar/api-reference")

    @app.get("/scalar", include_in_schema=False)
    async def scalar_html(request: Request):
        root_path = request.scope.get("root_path", "").rstrip("/")
        openapi_url = root_path + _openapi_url
        scalar_js_url = root_path + _scalar_js_url
        return get_scalar_ui_html(
            openapi_url=openapi_url,
            title=f"{_title} - Swagger UI",
            servers=_servers,
            init_oauth=_init_oauth,
            scalar_js_url=scalar_js_url,
            **kwargs
        )

```

## 3. css/js 文件
```python title="static/__init__.py" showLineNumbers
from pathlib import Path as LibPath

from fastapi import FastAPI, APIRouter, Path
from pydantic import BaseModel
from starlette.responses import FileResponse


class OfflineJsPath(BaseModel):
    scalar_ui_js: str
    swagger_ui_js: str
    swagger_ui_css: str


def get_offline_js_path(app: FastAPI):
    static_url = '/openapi'
    static_path = LibPath(__file__).parent
    router = APIRouter(prefix=static_url)

    @router.get('/scalar-ui{suffix}', include_in_schema=False)
    async def get_scalar_ui(suffix: str = Path(...)):
        return FileResponse(static_path / f'scalar-ui{suffix}')

    @router.get('/swagger-ui{suffix}', include_in_schema=False)
    async def get_swagger_ui(suffix: str = Path(...)):
        return FileResponse(static_path / f'swagger-ui{suffix}')

    app.include_router(router)

    return OfflineJsPath(
        scalar_ui_js=f'{static_url}/scalar-ui.js',
        swagger_ui_js=f'{static_url}/swagger-ui-bundle.js',
        swagger_ui_css=f'{static_url}/swagger-ui.css'
    )


```
:::note[注意]
如果需要离线使用 Swagger UI 和 Scalar UI，需要将 `swagger-ui-bundle.js`, `swagger-ui.css`, `scalar.js` 文件放到 `static` 下（与__init__.py同级）。
> static/swagger-ui-bundle.js
>
> static/swagger-ui.css
>
> static/scalar.js
>
> static/\__init__.py
:::

## 4. 使用示例

```python file="main.py" showLineNumbers
# main.py
from fastapi import FastAPI
from swagger import swagger
from scalar import scalar
app = FastAPI(title="My API", version="1.0.0")

offline = openapi.get_offline_js_path(app)
openapi.scalar(app, hide_models=True, scalar_js_url=offline.scalar_ui_js)
openapi.swagger(app, swagger_js_url=offline.swagger_ui_js, swagger_css_url=offline.swagger_ui_css)

```
## 5. 访问地址

### Swagger UI
http://localhost:8000/swagger

### Scalar UI
http://localhost:8000/scalar
