Escuela Colombiana de Ingeniería
--------------------------------

#### Construcción de Software - COSW

#### Android básico – interoperabilidad y manejo de hilos.

### Parte I.

En este ejercicio va a implementar un cliente REST para el API disponible en
<https://products-catalog-api.herokuapp.com/products>, el cual soporta los
métodos GET (consultar TODOS los productos) y POST (registrar un nuevo
producto). Tenga en cuenta que este API tiene la protección contra
[CSRF](<https://es.wikipedia.org/wiki/Cross-site_request_forgery>) deshabilitada
(para permitir peticiones desde clientes externos). Si va a implementar un
cliente Android en su propio API, no olvide deshabilitarlo también en la
configuración de seguridad de SpringBoot:

```java
protected void configure(HttpSecurity http) throws Exception {
    http.
    ...
    .csrf().disable();
}
```

Para hacer este ejercicio, tenga como referencia los siguientes segmentos de
código:

#### Peticiones POST

Para enviar una petición POST con el siguiente objeto JSON:

```javascript
{“elementoUno”: “valorUno”, “elementoDos”,999999}

```

suponiendo que el recurso está en la URL https://LAURLDELRECURSO :

```java

JSONObject jso=new JSONObject();

jso.put("elementoUno","valor1");
jso.put("elementoDos",999999);

URL obj = new URL("https://LAURLDELRECURSO");
HttpsURLConnection con = (HttpsURLConnection) obj.openConnection();

con.setRequestMethod("POST");
con.setRequestProperty("Content-Type", "application/json");

OutputStream out = con.getOutputStream();

out.write(jso.toString().getBytes());

out.flush();
out.close();

//Mensaje de respuesta
String respmsg = ((HttpsURLConnection)con).getResponseMessage();
//Código HTTP de respuesta
int restcode=con.getResponseCode();

```
### Peticiones GET

Para enviar una petición GET a un recurso, suponiendo que el mismo está en la URL http://RESOURCEURL:

1.  Para hacer la petición y obtener la respusta en forma de texto plano:

	```java
    public String readResourceContent() {

		URL obj = new URL("https://products-catalog-api.herokuapp.com/products");
		HttpsURLConnection con = (HttpsURLConnection) obj.openConnection();
		
		con.setRequestMethod("GET");
		con.setRequestProperty("Content-Type", "application/json");
		con.setRequestProperty("Accept-Language", "en-US,en;q=0.5");
				
		int rc=con.getResponseCode();
		
		BufferedReader in = new BufferedReader(
		        new InputStreamReader(con.getInputStream()));
		String inputLine;
		StringBuffer response = new StringBuffer();
		
		while ((inputLine = in.readLine()) != null) {
		    response.append(inputLine);
		}
		in.close();
		
		return response.toString();


    }
```

2.  Para convertir la cadena de texto (suponiendo que la misma tiene formato
    JSON) en objetos que puedan ser procesados, revise el API de JSON para
    Android, en particular las clases
    [JSONObject](<http://developer.android.com/reference/org/json/JSONObject.html>)
    y
    [JSONArray](<http://developer.android.com/reference/org/json/JSONArray.html>)

Con lo anterior, una vez convertido el texto recibido de la petición a un objeto
JSONArray o JSONObject, puede acceder a una de las posiciones de un arreglo
jSON, o a un campo del objeto jSON (respectivamente).

### Seguridad

Si el API que va a usar tiene un esquema de autenticación básico, debe agregar
en los encabezados el usuario y contraseña (recuerde que este esquema sólo es
válido en producción si se está usando un canal seguro como HTTPS!). En el API
de Android, ésto lo puede hacer al momento de crear los objetos HttpsURLConnection.

```java

HttpsURLConnection con=...

String userpass = "user" + ":" + "user";
String basicAuth = "Basic " + new String(Base64.encodeToString(userpass.getBytes(), Base64.NO_WRAP));
con.setRequestProperty ("Authorization", basicAuth);

```

### Procesos asíncronos

Para efectos de garantizar la fluidez y consistencia de las aplicaciones,
Android tiene las siguientes restricciones:

-   No se pueden realizar operaciones de alta latencia (tales como peticiones a
    través de la red) en el hilo principal de la actividad.

-   No se puede manipular la vista de una actividad desde un hilo diferente al
    de su actividad correspondiente. Es decir, con el fin de consumir el
    servicio Web, la aplicación debe:

    -   Capturar los parámetros de la vista

    -   Desde el hilo principal, crear e iniciar un hilo secundario, el cual
        enviará la petición y esperará por la respuesta.

    -   Sincronizar el hilo principal con el hilo secundario, de manera que se
        identifique cuando terminó este último, y actualizar la vista con los
        resultados obtenidos.

Parte 1.
--------

Teniendo en cuenta todo lo anterior, se quiere crear una aplicación Android con
una Actividad que permita:

1.  Capturar los detalles de un nuevo producto (para esto haga una petición GET
    al API indiciado al comienzo del enunciado y revise qué propiedades tiene un
    producto).

2.  Al oprimir un botón, tomar los valores ingresados, crear un objeto jSON
    adecuado, y enviar éste a través de una petición POST. Adicionalmente,
    mostrar en la pantalla si la operación fue exitosa o no. Para lo anterior:

    -   Agregue a la vista tantos campos como propiedades tenga un producto, e
        incluya un botón para realizar la acción y una etiqueta para mostrar el
        resultado de la operación.

    -   Abra el manifiesto de Android, y agregue el ‘permiso de uso’ para
        Internet (la opción de autocompletar le mostrará la lista de opciones):
        \<uses-permission

    -   Cree una clase que herede de la clase genérica AsyncTask
        (http://developer.android.com/reference/android/os/AsyncTask.html ),
        usando los siguientes tipos para sustituir los genéricos:

        -   ‘Params’ -\> JSONObject : al AsyncTask se le enviará como parámetro
            el objeto jSON a ser enviado.

        -   ‘Progress’ -\> Integer : aunque no se usará, sirve para definir que
            el progreso de la tarea asíncrona se podrá cuantificar a través de
            un entero.

        -   ‘Result’ -\> String: el proceso asíncrono retornará el resultado de
            la operación como un String.

3.  En la clase creada, redefina los métodos:

    ```java
    @Override
    protected void onPreExecute() {
    //lo que se hace antes de crar el hilo secundario. En este caso, no es //necesario hacer nada.
    }

    @Override
    protected String doInBackground(JSONObject... params) {

        //En este método irá lo que se realizará en el hilo secundario.
        //En este caso, se debe realizar la petición HTTP, esperar por la respuesta, //y retornarla. Lo retornado será manejado por el método onPostExecute.
    }

    @Override
    protected void onPostExecute(String result) {
           //En este método se procesa el resultado del hilo secundario. 
            //En este caso, se debe actualizar la vista con el resultado del servicio Web.
            //Tenga en cuenta que para esto deben tener la referencia de la Actividad 
            //para poder hacer el ‘findViewById‘

    }   
```

4.  Una vez hecho esto, haga que el evento del botón creado:

    -   Consulte los campos diligenciados.

    -   Cree un objeto jSON.

    -   Cree una instancia del AsyncTask creado, e invocar en éste el método
        “execute”, pasándole como parámetro el objeto jSON.

5.  Agregue, donde corresponda, los encabezados para que el móvil se autentique
    ante el API, teniendo usando el usuario 'user' con contraseña 'user'.

6.  Verifique el funcionamiento de la aplicación (el móvil debe tener un plan de
    datos, o estar conectado a una red WiFi). Cree un nuevo producto y revise en
    un browser (a través de una petición GET) que el producto dado fue
    registrado.

Parte II.
---------

1.  Teniendo en cuenta que **ES INADMISIBLE** que una aplicación tenga
    'quemados' usuarios o contraseñas, haga que éstos puedan ser ingresados
    desde la actividad principal.

2.  Agregue una funcionalidad que permita consultar los productos disponibles, y
    que los muestre en una caja de texto, y verifique que funcione correctamente
    (tenga en cuenta que los datos obtenidos con la petición GET al recurso
    ‘products’ son de tipo JSONArray).

3.  Revise la documentación de
    [TableLayout](<http://developer.android.com/reference/android/widget/TableLayout.html>),
    y con la [documentación de cómo agregar filas dinámicamente a una
    tabla](<https://technotzz.wordpress.com/2011/11/04/android-dynamically-add-rows-to-table-layout/>)
    modifique la actividad para que en lugar de sólo mostrar el documento jSON
    muestre los detalles de los productos en forma de tabla.
