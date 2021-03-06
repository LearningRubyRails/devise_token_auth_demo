# Devise Token Auth Demo

### Para propositos de aprendizaje.


Instalamos las gemas:
~~~ruby
gem 'devise_token_auth'
gem 'omniauth'
~~~

Creamos la base de datos:
~~~ruby
rails db:create
~~~

Corremos el comando:
~~~ruby
rails g devise_token_auth:install User auth
~~~

Migramos:
~~~ruby
rails db:migrate
~~~

Si el API y el cliente van a estar en diferentes dominios, debes configurar tu aplicación para que acepte  cross origin requests:

Asegurate de permitir solo los dominios que tendran acceso al API.
~~~ruby
# gemfile
gem 'rack-cors', :require => 'rack/cors'

# config/initializers/cors.rb
Rails.application.config.middleware.insert_before 0, Rack::Cors do
  allow do
    origins '*'
    resource '*',
    :headers => :any,
    :expose  => ['access-token', 'expiry', 'token-type', 'uid', 'client'],
    :methods => [:get, :post, :options, :delete, :put]
  end
end
~~~

Para que no se regenere el token en cada request:
~~~ruby
# config/initializers/devise_token_auth.rb
config.change_headers_on_each_request = false
~~~

- En el modelo User no se puede validar el email y el password, ya que se validan por defecto.


Ejemplo de request para registrarse:

![Sign Up](https://github.com/learningruby/devise_token_auth_demo/blob/master/public/images/sign_up.png?raw=true)

Ejemplo de request para iniciar sesión:

![Sign In](https://github.com/learningruby/devise_token_auth_demo/blob/master/public/images/sign_in.png?raw=true)

Respuesta de inicio de sesion:

![Sign In Response](https://github.com/learningruby/devise_token_auth_demo/blob/master/public/images/sign_in_response.png?raw=true)

Headers devueltos en el inicio de sesión:

![Sign In Headers](https://github.com/learningruby/devise_token_auth_demo/blob/master/public/images/sign_in_headers.png?raw=true)


### Cambio de contraseña

Para que requiera la contraseña actual
~~~ruby
# config/initializers/devise_token_auth.rb
config.check_current_password_before_update = :password
~~~

### Recuperación de contraseña

Se realizo un fork del repositorio de [devise token auth](https://github.com/lynndylanhurley/devise_token_auth), el cambio quedo en el siguiente [repositorio con el fork](https://github.com/emmanuelsw/devise_token_auth).

Se tuvo la necesidad de realizar esto porque en la versión del repositorio original si se requiere la contraseña actual para cambiar la contraseña no es posible la recuperación sin este campo.

Se agrego el campo  **allow_password_change** al modelo de usuarios como un campo **boolean** y por defecto **false**


### Action Cable

Por defecto action cable viene configurado para solo recibir peticiones del mismo servidor, para cambiar esto se hace lo siguiente en cada ambiente que se ejecute, en este caso se mostrara en ambiente de desarrollo.

~~~ruby
# config/environments/development.rb
config.action_cable.allowed_request_origins = [/http:\/\/*/, /https:\/\/*/]
~~~

Se creo la funcion de action cable cuando se crea un nuevo producto, para esto debemos crear el canal de escucha.

~~~ruby
rails g channel product
~~~

En el metodo **subscribed**, le decimos que se conectada a un canal llamado **product**

~~~ruby
# /app/channels/product_channel.rb
class ProductChannel < ApplicationCable::Channel
	def subscribed
		stream_from "product"
	end

	def unsubscribed
		
	end
end
~~~

Ya con esto tenemos un canal de escucha, solo falta crear una accion que ejecute el canal para enviar a todos los usuarios que se encuentren suscrito a el informacion, para esto vamos al modelo de producto y creamos un metodo para que se ejecute luego de que un producto es creado.

~~~ruby
# /app/models/product.rb
class Product < ApplicationRecord
	.
	.
	.
	after_create :send_to_action_cable

	.
	.
	.
	private
		def send_to_action_cable
			data = {message: self,action:"new product"}
			ActionCable.server.broadcast "product",data
		end
end
~~~



