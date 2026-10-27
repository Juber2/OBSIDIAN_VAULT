ip servidor: 144.202.33.102
Clave servidor: oUo1w^N,mCxk0r(;

usuario base de datos postgre
user: app_gastos_user
pass: oUo1w^N,mCxk0r(

ALTER ROLE postgres WITH LOGIN SUPERUSER PASSWORD 'oUo1w^N,mCxk0r(';

abrir tunel 
ssh -L 5433:127.0.0.1:5432 jt@144.202.33.102

loguearse a bd postgre
 docker exec -it app-control-gastos-db psql -U postgres -d app_control_gastos -W

Tareas a realizar: 
Cambiar el color de las calificaciones (estrellas) a amarillo.
En la sección Productos, organizar los productos en 3 columnas en lugar de 4. 
Cambiar la forma del cursor como prefieras, no como un circulo. 
Al pasar el cursor sobre los productos, agregar una animacion de zoom enmascarada de arriba hacia abajo con color gris claro.





