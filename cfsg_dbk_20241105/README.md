# Readme

Esta guía te ayudará a crear un sistema de inicio de sesión utilizando Laravel, con validaciones y respuestas estructuradas según los requisitos comunes para un examen o proyecto práctico.

---

## **1. Preparar el Entorno**

### **Instalar Laravel**

1. Crea un proyecto de Laravel:
   ```bash
   composer create-project --prefer-dist laravel/laravel examen_dbk
   ```
2. Accede al directorio del proyecto:
   ```bash
   cd examen_dbk
   ```

### **Configurar la Base de Datos**

1. Abre el archivo `.env` y ajusta los siguientes parámetros:
2. ejecute uno de los siguientes comandos:
o composer create-project --prefer-dist laravel/laravel jjcf_dbk_20241105
o composer create-project laravel/laravel jjcf_dbk_20241105


   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=nombre_base_datos
   DB_USERNAME=tu_usuario
   DB_PASSWORD=tu_contraseña
   ```

3. Verifica la conexión y crea las tablas necesarias en tu base de datos.

   - Si no existen migraciones, crea una para la tabla `users`:
     ```bash
     php artisan make:migration create_users_table --create=users
     ```
   - Agrega las columnas requeridas:
     ```php
     public function up()
     {
         Schema::create('users', function (Blueprint $table) {
             $table->id();
             $table->string('name', 100);
             $table->string('email', 100)->unique();
             $table->string('password');
             $table->timestamps();
         });
     }
     ```
   - Aplica las migraciones:
     ```bash
     php artisan migrate
     ```

### **Instalar Laravel Sanctum**

Laravel Sanctum se utilizará para manejar la autenticación basada en tokens.

1. Instala Sanctum:

   ```bash
   composer require laravel/sanctum
   php artisan vendor:publish --provider="Laravel\\Sanctum\\SanctumServiceProvider"
   php artisan migrate
   ```

2. Configura el middleware en `app/Http/Kernel.php`:

   ```php
   'api' => [
       \Laravel\Sanctum\Http\Middleware\EnsureFrontendRequestsAreStateful::class,
       'throttle:api',
       \Illuminate\Routing\Middleware\SubstituteBindings::class,
   ],
   ```

---

## **2. Crear el Controlador para la Autenticación**

### **Generar un Controlador**

Crea un controlador para manejar las funciones de registro, inicio de sesión y detalles del usuario:

```bash
php artisan make:controller AuthController
```

### **Implementar Funciones en el Controlador**

Agrega las siguientes funciones en `AuthController`:

#### **Registrar Usuario (`register`)**

```php
public function register(Request $request)
{
    $validatedData = $request->validate([
        'name' => ['required', 'string', 'max:100'],
        'email' => ['required', 'string', 'email', 'max:100', 'unique:users'],
        'password' => ['required', 'string', 'min:8', 'max:20'],
    ]);

    $user = User::create([
        'name' => $validatedData['name'],
        'email' => $validatedData['email'],
        'password' => Hash::make($validatedData['password']),
    ]);

    $token = $user->createToken('auth_token')->plainTextToken;

    return response()->json([
        "success" => true,
        "errors" => ["code" => 0, "msg" => ""],
        "data" => ["access_token" => $token, "token_type" => "Bearer"],
        "msg" => "Usuario creado satisfactoriamente",
        "count" => 1,
    ], 201);
}
```

#### **Inicio de Sesión (`login`)**

```php
public function login(Request $request)
{
    if (!Auth::attempt($request->only('email', 'password'))) {
        return response()->json([
            "success" => false,
            "errors" => ["code" => 401, "msg" => "No se reconocen las credenciales"],
            "data" => "",
            "count" => 0,
        ], 401);
    }

    $user = User::where('email', $request->email)->firstOrFail();
    $token = $user->createToken('auth_token')->plainTextToken;

    return response()->json([
        "success" => true,
        "errors" => ["code" => 200, "msg" => ""],
        "data" => ["access_token" => $token, "token_type" => "Bearer"],
        "count" => 1,
    ]);
}
```

#### **Obtener Datos del Usuario Autenticado (`me`)**

```php
public function me(Request $request)
{
    return response()->json([
        "success" => true,
        "errors" => ["code" => 200, "msg" => ""],
        "data" => $request->user(),
        "count" => 1,
    ]);
}
```

---

## **3. Configurar las Rutas**

Agrega las siguientes rutas en `routes/api.php`:

```php
use App\Http\Controllers\AuthController;

Route::post('/nuevousuario', [AuthController::class, 'register']);
Route::post('/login', [AuthController::class, 'login']);
Route::post('/usuario', [AuthController::class, 'me'])->middleware('auth:sanctum');
```

---

## **4. Probar los Endpoints**

### **Registro (`POST /nuevousuario`)**

- JSON de entrada:
  ```json
  {
      "name": "John Doe",
      "email": "johndoe@example.com",
      "password": "password123"
  }
  ```

### **Inicio de Sesión (`POST /login`)**

- JSON de entrada:
  ```json
  {
      "email": "johndoe@example.com",
      "password": "password123"
  }
  ```

### **Obtener Datos del Usuario Autenticado (`POST /usuario`)**

- Agregar el encabezado:
  ```
  Authorization: Bearer <tu_token_aqui>
  ```

---

## **5. Resolución de Problemas**

1. **Migraciones Faltantes:**

   - Ejecuta: `php artisan migrate`.

2. **Token Inválido:**

   - Verifica que el encabezado `Authorization` incluya un token válido.

3. **Errores de Validación:**

   - Asegúrate de enviar todos los campos requeridos en el formato correcto.

---

Con esta guía, estarás preparado para implementar un sistema de login funcional en Laravel para tu examen o proyecto. Si tienes dudas o necesitas ayuda adicional, no dudes en consultar.

