# Comandos wpcli
Colección de comandos utilies para ejecutar cli

Instalar la tienda por primera vez:
```bash
docker compose run --rm wpcli wp core install \
  --url="http://localhost:8080" --title="Mi tienda" \
  --admin_user=admin --admin_password=admin --admin_email=leviatan.info@gmail.com
```
Instalar el plugin woocommerce:
```bash
docker compose run --rm wpcli wp plugin install woocommerce --activate
```

Instalar el plugin novamira (no está en el repositorio oficial de WordPress, se instala desde ZIP):
```bash
docker compose run --rm wpcli wp plugin install "https://github.com/use-novamira/novamira/releases/download/v1.12.8/novamira-1.12.8.zip" --activate
```
Versión más reciente (siempre última):
```bash
docker compose run --rm wpcli wp plugin install "https://license.dynamic.ooo/api/novamira/download" --activate
```
Verificar que el plugin quedó activo:
```bash
docker compose run --rm wpcli wp plugin list
```

Requisitos novamira:
- WordPress 6.9+ ( Abilities API del core ): `docker compose run --rm wpcli wp core version`
- PHP 8.0+ ( la imagen `wordpress:latest` ya lo cumple )
- Node.js NO se instala en el contenedor: solo se necesita en la máquina local ( Windows/WSL ) para el transporte MCP remoto. Con WP-CLI/STDIO no aplica.
- La instalación no habilita las habilidades de IA por defecto: se configuran en el dashboard ( Novamira > Configuration ) o por `wp option update`.

## ngrok ( dominio público para conectar el LLM vía MCP )

La instalación se mantiene con `--url="http://localhost:8080"`: el snippet en `./app/wp-config.php` hace que WordPress use el dominio ngrok como URL canónica solo cuando el request viene de ngrok ( el subdominio free cambia en cada reinicio, no hay que actualizar nada ):

```php
// URLs canónicas: ngrok cuando se accede por ngrok, localhost en local
if ( isset( $_SERVER['HTTP_HOST'] ) && str_contains( $_SERVER['HTTP_HOST'], 'ngrok' ) ) {
	define( 'WP_HOME', 'https://' . $_SERVER['HTTP_HOST'] );
	define( 'WP_SITEURL', 'https://' . $_SERVER['HTTP_HOST'] );
}
```

- El HTTPS tras ngrok ya lo detecta el propio `wp-config.php` del contenedor ( header `X-Forwarded-Proto` ), no requiere cambios extra: las Application Passwords quedan habilitadas.
- Abrir el túnel ( en WSL ): `ngrok http 8080`
- Crear una Application Password para el MCP ( la usa el AI client / proxy Novamira ):
```bash
docker compose run --rm wpcli wp user application-password create admin "mcp" --porcelain
```
`--porcelain` imprime solo la contraseña de aplicación: guárdala para configurar la conexión en Novamira / tu AI client.
