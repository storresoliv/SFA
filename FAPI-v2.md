### Prólogo

La OpenID Foundation (OIDF) promueve, protege y fomenta la comunidad y las tecnologías de OpenID. Como organismo internacional sin fines de lucro dedicado a la estandarización, está compuesto por más de 160 entidades participantes (participantes en el grupo de trabajo). El trabajo de preparar borradores para implementadores y los estándares internacionales finales se lleva a cabo a través de los grupos de trabajo de OIDF, de acuerdo con el Proceso OpenID. Los participantes interesados en un tema para el cual se ha establecido un grupo de trabajo tienen el derecho de estar representados en ese grupo. Las organizaciones internacionales, tanto gubernamentales como no gubernamentales, en contacto con la OIDF, también participan en el trabajo. OIDF colabora estrechamente con otros organismos de estandarización en los campos relacionados.

Los borradores finales adoptados por el grupo de trabajo mediante consenso se distribuyen públicamente para su revisión pública durante 60 días y para la votación de los miembros de OIDF. La publicación como un estándar de OIDF requiere la aprobación de al menos el 50% de los miembros que emitan su voto. Existe la posibilidad de que algunos de los elementos de este documento puedan estar sujetos a derechos de patente. OIDF no se hace responsable de identificar dichos derechos de patente, ya sea en su totalidad o en parte.

### Introducción

El Perfil de Seguridad FAPI 2.0 es un perfil de seguridad para API basado en el Marco de Autorización OAuth 2.0 [RFC6749] y especificaciones relacionadas, que tiene como objetivo alcanzar los objetivos de seguridad establecidos en el Modelo de Atacante [attackermodel], para que sea adecuado para proteger APIs en escenarios de alto valor. También sigue las recomendaciones del BCP de Seguridad de OAuth [I-D.ietf-oauth-security-topics].

Este documento especifica el proceso para que un cliente obtenga tokens con restricciones de remitente desde un servidor de autorización y los utilice de manera segura con los servidores de recursos.

El Grupo de Trabajo FAPI de la OpenID Foundation publica documentos adicionales que se basan en este perfil como parte del marco FAPI 2.0.

La propiedad de seguridad es analizada formalmente [FAPI2SEC] bajo el modelo de atacante mencionado anteriormente. Para las suposiciones de seguridad, consulte el modelo de atacante.

Aunque el perfil de seguridad se desarrolló inicialmente con un enfoque en aplicaciones financieras, está diseñado para ser aplicable universalmente para proteger APIs que exponen datos de alto valor y sensibles (personales y otros), por ejemplo, en aplicaciones de e-salud y e-gobierno.

### Advertencia

Este documento no es un Estándar Internacional de OIDF. Se distribuye para su revisión y comentarios. Está sujeto a cambios sin previo aviso y no debe ser referido como un Estándar Internacional.

Se invita a los destinatarios de este borrador a enviar, junto con sus comentarios, notificaciones sobre cualquier derecho de patente relevante del que tengan conocimiento y a proporcionar la documentación de apoyo correspondiente.

### Convenciones de notación

Las palabras clave "shall" (deberá), "shall not" (no deberá), "should" (debería), "should not" (no debería), "may" (puede) y "can" (puede) en este documento deben interpretarse según lo descrito en la Parte 2 de la Directiva ISO [ISODIR2]. Estas palabras clave no se utilizan como términos del diccionario, por lo que cualquier aparición de ellas debe interpretarse como palabras clave y no con sus significados en lenguaje natural.

### 1. Alcance

Este documento proporciona un perfil de alta seguridad de propósito general de OAuth 2.0 que ha sido validado mediante análisis formal para cumplir con el modelo de atacante establecido. Este documento especifica los requisitos para:

- Clientes confidenciales para obtener de forma segura tokens de OAuth de los servidores de autorización;
- Clientes confidenciales para utilizar esos tokens de forma segura para acceder a recursos protegidos en servidores de recursos;
- Servidores de autorización para emitir de manera segura tokens de OAuth a clientes confidenciales;
- Servidores de recursos para aceptar y verificar de forma segura los tokens de OAuth de los clientes confidenciales.

### 2. Referencias normativas

Los siguientes documentos se citan en el texto de tal manera que parte o la totalidad de su contenido constituyen requisitos de este documento. Para las referencias con fecha, solo se aplica la edición citada. Para las referencias sin fecha, se aplica la última edición del documento referenciado (incluidas las enmiendas).

Consulte la Sección 11 para las referencias normativas.

### 3. Términos y definiciones

A efectos de este documento, se aplican los términos definidos en [RFC6749], [RFC6750], [RFC7636], [OIDC] y [ISO29100].

### 4. Símbolos y términos abreviados

- API – Application Programming Interface

- BCM – Basin, Cremers, Meier

- BCP – Best Current Practice

- CAA – Certificate Authority Authorization

- CIBA – Client Initiated Backchannel Authentication

- CSRF – Cross-Site Request Forgery

- DNS – Domain Name System

- DNSSEC – Domain Name System Security Extensions

- HTTP – Hyper Text Transfer Protocol

- JAR – JWT-Secured Authorization Request

- JARM – JWT Secured Authorization Response Mode

- JWK – JSON Web Key

- JWKS – JSON Web Key Sets

- JWT – JSON Web Token

- JOSE – Javascript Object Signing and Encryption

- JSON – JavaScript Object Notation

- MTLS – Mutual Transport Layer Security

- OIDF – OpenID Foundation

- PAR – Pushed Authorization Requests

- PKCE – Proof Key for Code Exchange

- QR – Quick Response

- RSA – Rivest-Shamir-Adleman

- REST – Representational State Transfer

- TLS – Transport Layer Security

- URI – Uniform Resource Identifier

- URL – Uniform Resource Locator

### 5. Perfil de seguridad

#### 5.1. Visión general

##### 5.1.1. Introducción

El Perfil de Seguridad FAPI 2.0 es un perfil de seguridad para API basado en el Marco de Autorización OAuth 2.0 [RFC6749], que tiene como objetivos:  
- Alcanzar los objetivos de seguridad establecidos en el Modelo de Atacante [attackermodel]; y  
- Seguir las recomendaciones del BCP de Seguridad de OAuth [I-D.ietf-oauth-security-topics].

El Grupo de Trabajo FAPI de OpenID no tiene actualmente conocimiento de mecanismos que permitan asegurar a los clientes públicos en el mismo grado, por lo que su uso no está dentro del alcance de este documento.

Aunque es posible desarrollar servidores de autorización y clientes desde los principios básicos utilizando este documento, se recomienda a los implementadores basar sus desarrollos en implementaciones existentes de OpenID Connect y/o OAuth 2.0, en lugar de comenzar una implementación "desde cero". Consulte la Sección 6.6 para consideraciones adicionales sobre cómo garantizar que las implementaciones sean completas y correctas.

##### 5.1.2. Perfilado de este documento

Este documento es un perfil de alta seguridad de propósito general de OAuth 2.0 que ha sido validado mediante análisis formal para cumplir con el modelo de atacante establecido.

Este documento, y las especificaciones subyacentes, dejan abiertas varias opciones para los implementadores, desplegadores y/o ecosistemas. Conociendo los casos de uso exactos, reducir aún más el número de opciones podría mejorar la seguridad, o facilitar la implementación o la interoperabilidad.

Sin embargo, para que un perfil cumpla con este documento, no deberá eliminar ni anular los comportamientos obligatorios, ya que hacerlo probablemente invalidaría el análisis de seguridad formal y reduciría la seguridad de manera potencialmente impredecible.

### 5.2. Protecciones en la capa de red

#### 5.2.1. Requisitos para todos los puntos finales

Para protegerse contra ataques en la red, los clientes, servidores de autorización y servidores de recursos:

- **deberán** ofrecer únicamente puntos finales protegidos por TLS y **deberán** establecer conexiones con otros servidores utilizando TLS;
- **deberán** configurar las conexiones TLS usando la versión 1.2 o posterior de TLS;
- **deberán** seguir las recomendaciones para el uso seguro de la Seguridad en la Capa de Transporte en [BCP195];
- **deberían** utilizar DNSSEC para proteger contra ataques de suplantación de DNS que puedan conducir a la emisión de certificados TLS válidos por dominios falsificados;
- **deberán** realizar una verificación del certificado del servidor TLS, conforme a [RFC6125].

**NOTA 1**: Incluso si un punto final utiliza únicamente certificados TLS validados por la organización (OV) o validación extendida (EV), un atacante que use certificados de dominio falso puede suplantar el punto final y llevar a cabo ataques de "man-in-the-middle". Los registros CAA [RFC8659] ayudan a mitigar este riesgo.

#### 5.2.2. Requisitos para puntos finales no utilizados por navegadores web

Para los puntos finales de comunicación servidor a servidor que no son utilizados por navegadores web, se aplican los siguientes requisitos:

- Cuando se utilice TLS 1.2, los servidores **deberán** permitir únicamente los conjuntos de cifrado recomendados en [BCP195];
- Cuando se utilice TLS 1.2, los clientes **deberían** permitir únicamente los conjuntos de cifrado recomendados en [BCP195].

#### 5.2.2.1. Ecosistemas MTLS

Algunos ecosistemas pueden implementar MTLS como un control de seguridad adicional en la capa de transporte para todos los puntos finales servidor a servidor que requieran la transmisión de datos sensibles. Por ejemplo, **private_key_jwt** puede ser utilizado para la autenticación de clientes junto con la conectividad MTLS. Para facilitar la interoperabilidad:

- Los ecosistemas MTLS **deberían** proporcionar la lista de confianza de las autoridades de certificación;
- Las implementaciones de servidores de autorización **podrán** utilizar el metadato de servidor de autorización **mtls_endpoint_aliases** como se describe en la Sección 5 de [RFC8705] para proporcionar un mecanismo de descubrimiento para los puntos finales que puedan tener tanto puntos finales MTLS como no MTLS;
- Las implementaciones de clientes **deberán** utilizar el metadato de cliente **use_mtls_endpoint_aliases** (según se define en la Sección 8 de este documento), si está presente, para las comunicaciones de puntos finales.

#### 5.2.3. Requisitos para puntos finales utilizados por navegadores web

Para los puntos finales utilizados por navegadores web, se aplican los siguientes requisitos adicionales:

- Los servidores **deberán** utilizar métodos para garantizar que las conexiones no puedan ser degradadas mediante ataques de TLS stripping. Para este propósito, se puede utilizar una política de **HTTP Strict Transport Security** pre-cargada [preload] [RFC6797]. Algunos dominios de nivel superior, como .bank y .insurance, han establecido dicha política, por lo que protegen todos los dominios de segundo nivel por debajo de ellos;
- Cuando se utilice TLS 1.2, los servidores **deberán** usar únicamente los conjuntos de cifrado permitidos en [BCP195];
- Los servidores **no deberán** admitir CORS [CORS.Protocol] para el punto final de autorización, ya que los clientes deben realizar una redirección HTTP en lugar de acceder directamente a este punto final.

**NOTA 1**: Al utilizar TLS 1.2, los puntos finales utilizados por navegadores web pueden usar cualquier conjunto de cifrado permitido en [BCP195], mientras que los puntos finales no utilizados por navegadores web solo pueden usar los conjuntos de cifrado recomendados en [BCP195].

**NOTA 2**: Se publicarán nuevas versiones de [BCP195] periódicamente por la IETF. Como mínimo, se espera que los implementadores cumplan con las versiones recién emitidas de BCP195 en un plazo de 12 meses o menos.

### 5.3. Perfil

#### 5.3.1. General

A continuación, se define un perfil de las siguientes tecnologías:

- Marco de Autorización OAuth 2.0 [RFC6749]
- Tokens de portador OAuth 2.0 [RFC6750]
- Prueba de clave para el intercambio de código por clientes públicos de OAuth (PKCE) [RFC7636]
- Autenticación mutua TLS de cliente OAuth 2.0 y tokens de acceso vinculados a certificados (MTLS) [RFC8705]
- Demostración de prueba de posesión de OAuth 2.0 (DPoP) [RFC9449]
- Solicitudes de autorización enviadas por OAuth 2.0 (PAR) [RFC9126]
- Metadatos del servidor de autorización OAuth 2.0 [RFC8414]
- Identificación del emisor del servidor de autorización OAuth 2.0 [RFC9207]
- OpenID Connect Core 1.0 con erratas del conjunto 1 [OIDC]

#### 5.3.2. Requisitos para los servidores de autorización

##### 5.3.2.1. Requisitos generales

Los servidores de autorización:

- **deberán** distribuir metadatos de descubrimiento (como el punto final de autorización) a través del documento de metadatos como se especifica en [OIDD] y [RFC8414];
- **deberán** rechazar solicitudes que utilicen el grant de credenciales de contraseña del propietario de los recursos;
- **deberán** soportar únicamente clientes confidenciales tal como se define en [RFC6749];
- **deberán** emitir únicamente tokens de acceso restringidos por remitente;
- **deberán** usar uno de los siguientes métodos para los tokens de acceso restringidos por remitente:
  - MTLS como se describe en [RFC8705],
  - DPoP como se describe en [RFC9449];
- **deberán** autenticar a los clientes usando uno de los siguientes métodos:
  - MTLS como se especifica en la Sección 2 de [RFC8705], o
  - **private_key_jwt** como se especifica en la Sección 9 de [OIDC];
- **no deberán** exponer redireccionadores abiertos (ver Sección 4.11 de [I-D.ietf-oauth-security-topics]);
- **deberán** aceptar el valor del identificador del emisor (como se define en [RFC8414]) ya sea como el claim *aud* (cuando sea una cadena) o como miembro del claim *aud* (cuando sea un array) recibido en las afirmaciones de autenticación del cliente;
- **deberían** aceptar la URL de su punto final de tokens o la URL del punto final en el que se recibió la afirmación, ya sea como el claim *aud* (cuando sea una cadena) o como miembro del claim *aud* (cuando sea un array) recibido en las afirmaciones de autenticación del cliente;
- **no deberán** usar rotación de tokens de actualización, excepto en circunstancias extraordinarias (ver Nota 2 a continuación);
- si se utiliza DPoP, **podrán** usar el mecanismo de nonce proporcionado por el servidor (como se define en la Sección 8 de [RFC9449]);
- **deberán** emitir códigos de autorización con una vida máxima de 60 segundos;
- si se utiliza DPoP, **deberán** soportar "Vinculación del Código de Autorización a la Clave DPoP" (según se requiere en la Sección 10.1 de [RFC9449]);
- para acomodar los desfases de reloj, **deberán** aceptar JWTs con un timestamp *iat* o *nbf* entre 0 y 10 segundos en el futuro, pero **deberán** rechazar JWTs con un timestamp *iat* o *nbf* mayor a 60 segundos en el futuro. Ver Nota 4 para más detalles y justificación;
- **deberían** restringir los privilegios asociados con un token de acceso al mínimo necesario para la aplicación o caso de uso particular.

**NOTA 1**: Para facilitar la interoperabilidad, este documento requiere que los servidores de autorización acepten su valor de emisor en el claim *aud* recibido en las afirmaciones de autenticación del cliente. Se recomienda que también acepten la URL de su punto final de tokens o la URL del punto final en el que se recibió la afirmación. Esto no reduce el requisito más estricto de [RFC9126], que requiere que los 3 valores sean aceptados en el punto final de PAR.

**NOTA 2**: El uso de la rotación de tokens de actualización no proporciona beneficios de seguridad cuando se utiliza con clientes confidenciales y tokens de acceso restringidos por remitente. Esta especificación prohíbe el uso de la rotación de tokens de actualización por razones de seguridad, ya que causa degradación de la experiencia del usuario y problemas operativos cuando el cliente no puede almacenar o recibir el nuevo token de actualización y no tiene opción de reintentar.

Sin embargo, dado que la rotación de tokens de actualización puede ser necesaria de vez en cuando por migraciones de infraestructura o circunstancias extraordinarias similares, esta especificación la permite, siempre que los servidores de autorización ofrezcan a los clientes la opción de reintentar con el token de actualización anterior en caso de fallo. Los implementadores deben considerar un mecanismo seguro para que los clientes se recuperen en caso de pérdida del nuevo token de actualización. Los detalles de este mecanismo están fuera del alcance de esta especificación.

**NOTA 3**: Este documento está estructurado para soportar una variedad de grants que se pueden utilizar con los requisitos generales anteriores. Por ejemplo, el grant de credenciales de cliente o el grant FAPI CIBA [FAPICIBA]. Los implementadores deben tener en cuenta que, al momento de escribir este documento, solo el flujo de código de autorización y los flujos CIBA han pasado por un análisis de seguridad detallado [FAPI2SEC].

**NOTA 4**: El desfase de reloj es una causa de muchos problemas de interoperabilidad. Incluso unos pocos cientos de milisegundos de desfase pueden hacer que los JWTs sean rechazados por ser "emitidos en el futuro". La especificación DPoP [RFC9449] sugiere que los JWTs sean aceptados en un futuro razonablemente cercano (en el orden de segundos o minutos). Este documento va más allá, exigiendo que los servidores de autorización acepten JWTs con timestamps hasta 10 segundos en el futuro. Se eligió el valor de 10 segundos como un valor que no afecta la seguridad, pero que aumenta considerablemente la interoperabilidad. Los implementadores son libres de aceptar JWTs con un timestamp de hasta 60 segundos en el futuro. Algunos ecosistemas han encontrado que se necesita un valor de 30 segundos para eliminar completamente los problemas de desfase de reloj. Para evitar que las implementaciones desactiven completamente las verificaciones *iat* y *nbf*, este documento impone un máximo de 60 segundos para el timestamp en el futuro.

### 5.3.2.2. Flujos del punto final de autorización

Para los flujos que utilizan el punto final de autorización, los servidores de autorización:

- **deberán** requerir el valor de *response_type* descrito en [RFC6749] como *code*;
- **deberán** soportar solicitudes de autorización enviadas por cliente autenticado, conforme a [RFC9126];
- **deberán** rechazar solicitudes de autorización enviadas sin cumplir con [RFC9126];
- **deberán** rechazar las solicitudes de autorización enviadas por cliente sin autenticación;
- **deberán** requerir el uso de PKCE [RFC7636] con S256 como el método de desafío de código;
- **deberán** requerir el parámetro *redirect_uri* en las solicitudes de autorización enviadas por cliente;
- **deberán** devolver el parámetro *iss* en la respuesta de autorización conforme a [RFC9207];
- **no deberán** transmitir respuestas de autorización a través de conexiones de red no cifradas, y para este fin, **no deberán** permitir URIs de redirección que usen el esquema "http", excepto para clientes nativos que usen la redirección de interfaz de bucle inverso como se describe en la Sección 7.3 de [RFC8252];
- **deberán** rechazar un código de autorización (Sección 1.3.1 de [RFC6749]) si ha sido utilizado previamente;
- **no deberán** usar el código de estado HTTP 307 cuando redirijan una solicitud que contenga credenciales de usuario, para evitar el envío accidental de las credenciales a un tercero (ver Sección 4.12 de [I-D.ietf-oauth-security-topics]);
- **deberían** usar el código de estado HTTP 303 cuando redirijan al agente de usuario mediante códigos de estado;
- **deberán** emitir solicitudes de autorización enviadas por cliente *request_uri* con valores *expires_in* de menos de 600 segundos;
- **deberían** proporcionar a los usuarios finales toda la información necesaria para tomar una decisión informada sobre si consienten la solicitud de autorización, incluida la identidad del cliente y el alcance de la autorización;
- si soporta [OIDC], **deberá** aceptar valores del parámetro *nonce* de hasta 64 caracteres de longitud, **podrá** rechazar valores de *nonce* que superen los 64 caracteres.

**NOTA 1**: Si no es posible la identificación de repetición del código de autorización, es recomendable establecer el periodo de validez del código de autorización a un minuto o a un periodo corto adecuado. El periodo de validez puede actuar como un indicador de control de caché sobre cuándo limpiar la caché del código de autorización, en caso de que se use.

**NOTA 2**: El tiempo *expires_in* de *request_uri* debe ser suficiente para que el dispositivo del usuario reciba el enlace y el usuario complete el proceso de apertura del enlace. En muchos casos (conexión de red deficiente o donde el usuario tiene que seleccionar manualmente el navegador a usar) esto puede tardar fácilmente más de 30 segundos.

**NOTA 3**: Se recomienda que los servidores de autorización que exigen el uso único de los valores de *request_uri* aseguren que esta exigencia se cumpla en el punto de autorización, no en el punto de carga de la página de autorización. Esto previene que el software del usuario, que precarga URLs, invalide *request_uri*.

**NOTA 4**: En este documento, el parámetro *state* no se usa para la protección contra CSRF, pero **puede** ser utilizado por el cliente para el estado de la aplicación. En circunstancias donde los clientes codifican el estado de la aplicación en un JWT, la longitud del valor del parámetro *state* podría exceder los 1000 caracteres.

**NOTA 5**: Se recomienda el uso de *OAuth 2.0 Rich Authorization Requests (RAR)* [RFC9396] cuando el parámetro *scope* no sea lo suficientemente expresivo para transmitir la autorización que un cliente pueda querer obtener.

### 5.3.2.3. Devolución del identificador del usuario autenticado

Si se desea proporcionar el identificador del usuario autenticado al cliente en la respuesta de token, el servidor de autorización **deberá** soportar OpenID Connect [OIDC].

### 5.3.3. Requisitos para los clientes

#### 5.3.3.1. Requisitos generales

Los clientes:

- **deberán** soportar tokens de acceso restringidos al remitente usando uno o ambos de los siguientes métodos:
  - MTLS, como se describe en [RFC8705],
  - DPoP, como se describe en [RFC9449];
  
- **deberán** soportar autenticación de clientes usando uno o ambos de los siguientes métodos:
  - MTLS, como se especifica en la Sección 2 de [RFC8705],
  - private_key_jwt, como se especifica en la Sección 9 de [OIDC];
  
- **deberán** enviar los tokens de acceso en el encabezado HTTP como en la Sección 2.1 de *OAuth 2.0 Bearer Token Usage* [RFC6750] o en la Sección 7.1 de DPoP [RFC9449];

- **no deberán** exponer redirigidores abiertos (ver Sección 4.11 de [I-D.ietf-oauth-security-topics]);

- Si usan *private_key_jwt*, **deberán** utilizar el valor del identificador del emisor del servidor de autorización (según lo definido en [RFC8414]) en el reclamo *aud* en las aserciones de autenticación del cliente, y **deberían** enviar el valor del identificador del emisor como una cadena, no como un elemento en un array;

- **deberán** soportar tokens de actualización (*refresh tokens*) y su rotación;

- Si usan autenticación de cliente MTLS o tokens de acceso restringidos al remitente con MTLS, **deberán** soportar los metadatos *mtls_endpoint_aliases* definidos en [RFC8705];

- Si usan DPoP, **deberán** soportar el mecanismo de nonce proporcionado por el servidor (como se define en la Sección 8 de [RFC9449]);

- **deberán** usar únicamente los metadatos del servidor de autorización (como el punto final de autorización) recuperados del documento de metadatos, tal como se especifica en [OIDD] y [RFC8414];

- **deberán** asegurarse de que la URL del emisor utilizada como base para obtener los metadatos del servidor de autorización provenga de una fuente autorizada y utilizando un canal seguro, de modo que no pueda ser modificada por un atacante;

- **deberán** asegurarse de que esta URL del emisor y el valor del emisor en los metadatos obtenidos coincidan;

- **deberán** iniciar un proceso de autorización solo con el consentimiento explícito o implícito del usuario final y proteger la iniciación de dicho proceso contra ataques de *cross-site request forgery (CSRF)*, permitiendo que el usuario final sea consciente del contexto en el que se inició el flujo;

- **deberían** solicitar la autorización con los mínimos privilegios necesarios para la aplicación o caso de uso específico.

**NOTA 1**: Este perfil puede ser utilizado por clientes confidenciales en un dispositivo controlado por el usuario, donde el reloj del sistema pueda no ser preciso, lo que causaría el fallo en la autenticación del cliente con *private_key_jwt*. En tales circunstancias, un cliente debería considerar el uso del encabezado HTTP *date* devuelto por el servidor para sincronizar su propio reloj al generar las aserciones de cliente.

**NOTA 2**: Aunque se requiere que los servidores de autorización soporten "Authorization Code Binding to DPoP Key" (como lo define la Sección 10.1 de [RFC9449]), no se requiere que los clientes lo utilicen.

#### 5.3.3.2. Flujo de código de autorización

Para el flujo de código de autorización, los clientes:

- **deberán** usar el flujo de código de autorización descrito en [RFC6749];
- **deberán** usar solicitudes de autorización enviadas por cliente según [RFC9126];
- **deberán** usar PKCE [RFC7636] con S256 como método de desafío de código;
- **deberán** generar el desafío PKCE específicamente para cada solicitud de autorización y vincularlo de manera segura al cliente y al agente de usuario en el que se inició el flujo;
- **deberán** verificar el parámetro *iss* en la respuesta de autorización según [RFC9207] para prevenir ataques de mezcla;
- **deberán** enviar únicamente los parámetros *client_id* y *request_uri* al punto final de autorización (todos los demás parámetros de la solicitud de autorización se envían en la solicitud de autorización enviada por cliente conforme a [RFC9126]);
- Si usan [OIDC], **deberían** no usar valores del parámetro *nonce* que sean más largos de 64 caracteres.

**NOTA 1**: Las restricciones recomendadas sobre la longitud del valor del parámetro *nonce* son para facilitar la interoperabilidad.

### 5.3.4. Requisitos para los servidores de recursos

Los puntos finales de FAPI 2.0 son puntos finales de recursos protegidos por OAuth 2.0 que devuelven información protegida para el propietario del recurso asociado con el token de acceso enviado.

Los servidores de recursos con los puntos finales de FAPI:

- **deberán** aceptar tokens de acceso en el encabezado HTTP como en la Sección 2.1 de *OAuth 2.0 Bearer Token Usage* [RFC6750] o en la Sección 7.1 de DPoP [RFC9449];
- **no deberán** aceptar tokens de acceso en los parámetros de consulta establecidos en la Sección 2.3 de *OAuth 2.0 Bearer Token Usage* [RFC6750];
- **deberán** verificar la validez, integridad, expiración y estado de revocación de los tokens de acceso;
- **deberán** verificar que la autorización representada por el token de acceso sea suficiente para el acceso al recurso solicitado y, en caso contrario, devolver errores como en la Sección 3.1 de [RFC6750];
- **deberán** soportar y verificar tokens de acceso restringidos al remitente usando uno o ambos de los siguientes métodos:
  - MTLS, como se describe en [RFC8705],
  - DPoP, como se describe en [RFC9449].

  ### 5.4. Criptografía y manejo de secretos

#### 5.4.1. Requisitos generales

Los siguientes requisitos se aplican a las operaciones criptográficas y al manejo de secretos:

- Los servidores de autorización, los clientes y los servidores de recursos, cuando creen o procesen JWTs, **deberán**:
  - Adherirse a [RFC8725];
  - Usar algoritmos **PS256**, **ES256**, o **EdDSA** (usando la variante Ed25519);
  - **No deberán** usar ni aceptar el algoritmo *none*.
  
- Las claves RSA **deberán** tener una longitud mínima de 2048 bits.

- Las claves de curva elíptica **deberán** tener una longitud mínima de 224 bits.

- Las credenciales que no están destinadas a ser manejadas por los usuarios finales (por ejemplo, tokens de acceso, tokens de actualización, códigos de autorización, etc.) **deberán** ser creadas con al menos 128 bits de entropía, de tal manera que adivinar el valor de forma correcta sea computacionalmente inviable. Cf. Sección 10.10 de [RFC6749].

**Nota**: Al momento de redactar este documento, no existe un algoritmo completamente registrado que describa "EdDSA usando la variante Ed25519". Si en el futuro se registra tal algoritmo, también podrá ser utilizado para este perfil.

#### 5.4.2. Conjuntos de claves JSON Web Key Sets (JWKS)

Este perfil **soporta** el uso de *private_key_jwt* y, además, permite el uso de OpenID Connect. Cuando se usan estas técnicas, los clientes y servidores de autorización necesitan verificar los payloads con claves de otra parte. Para los servidores de autorización, este perfil **recomienda fuertemente** el uso de puntos finales JWKS URI para distribuir claves públicas. Para la gestión de claves de los clientes, este perfil **recomienda** el uso de puntos finales JWKS URI o el uso del parámetro *jwks* en combinación con [RFC7591] y [RFC7592].

- La definición del *jwks_uri* del servidor de autorización se puede encontrar en [RFC8414], mientras que la definición del *jwks_uri* del cliente se puede encontrar en [RFC7591].

- Además, cualquier servidor que proporcione un punto final *jwks_uri*:
  - **Deberá** servir el punto final *jwks_uri* solo a través de TLS;
  - **No deberá** usar los encabezados JOSE *x5u* y *jku*;
  - **No deberá** servir un conjunto JWK con múltiples claves que tengan el mismo *kid*.

#### 5.4.3. Manejo de identificadores duplicados de claves (Duplicate Key Identifiers)

Los conjuntos JWK **deberían** no contener múltiples claves con el mismo *kid*. Sin embargo, para aumentar la interoperabilidad cuando haya múltiples claves con el mismo *kid*, el verificador **deberá** considerar otros atributos de JWK, como *kty*, *use*, *alg*, etc., al seleccionar la clave de verificación para el mensaje JWS particular. 

Por ejemplo, el siguiente algoritmo podría ser utilizado para seleccionar qué clave usar para verificar una firma de mensaje:

1. Buscar claves que tengan un *kid* que coincida con el *kid* en el encabezado JOSE;
2. Si se encuentra una única clave, usar esa clave;
3. Si se encuentran varias claves, el verificador **deberá** iterar entre las claves hasta encontrar una que tenga un *alg*, *use*, *kty*, o *crv* coincidente con el mensaje que se está verificando.

### 6. Consideraciones de seguridad

#### 6.1. Duración de los tokens de acceso

El uso de tokens de acceso de corta duración (combinado con tokens de actualización) puede reducir la ventana de tiempo en la que ciertos ataques pueden tener éxito.

- **Ventaja de los tokens de acceso cortos**: 
  - Limitan el tiempo en el que un token comprometido puede ser usado, mejorando la seguridad al restringir la exposición.
  - Reducen el impacto de compromisos o errores en la gestión de tokens, ya que los tokens caducan rápidamente.

- **Uso de tokens de actualización**:
  - Los tokens de actualización permiten a los clientes **rotar sus claves de restricción del remitente** sin perder los permisos, lo que puede ser útil si la clave es comprometida o como parte de una **buena higiene de seguridad** (renovación periódica de claves).
  
- **Consideraciones sobre la duración**:
  - Si se emiten permisos de larga duración (por ejemplo, días o semanas), se recomienda combinar tokens de acceso de corta duración (por ejemplo, minutos u horas) con **tokens de actualización** para mitigar el riesgo de exposición prolongada.

- **Impacto en el rendimiento y resiliencia**:
  - Configurar una duración de token demasiado corta puede **aumentar la carga** sobre el servidor de autorización, ya que los clientes necesitarían solicitar tokens de acceso nuevos con mayor frecuencia.
  - También puede aumentar la **dependencia** del cliente hacia el servidor de autorización, ya que los clientes necesitan renovar los tokens de acceso con mayor frecuencia.
  - Es importante balancear entre la seguridad (tokens de acceso más cortos) y la eficiencia operativa (evitar cargas innecesarias en el servidor).

  ### 6. Consideraciones de seguridad

#### 6.2. Repetición de pruebas DPoP

Un atacante de tipo A5 (ver [attackermodel]) podría obtener pruebas DPoP que podrían ser reutilizadas para realizar ataques.

- **Posible riesgo**: La reutilización de pruebas DPoP podría permitir a un atacante alterar una solicitud (por ejemplo, cambiar el monto de un pago o la cuenta de destino) y utilizar la prueba DPoP original.
  
- **Mitigaciones posibles**:
  - Los **servidores de recursos** pueden usar **nonces DPoP de corta duración** para reducir la ventana de tiempo en la que una solicitud puede ser reproducida.
  - Implementar prevención de repetición usando el **header `jti`** como se detalla en [RFC9449].
  - Para prevenir la repetición de una solicitud alterada, se pueden usar **solicitudes de recursos firmadas** como se especifica en **FAPI Message Signing** [FAPIMessageSigning].
  - Se podría considerar el uso de **MTLS** en lugar de DPoP para la restricción del remitente.
  
- **Consideraciones**: Estas mitigaciones pueden implicar trade-offs en cuanto a **complejidad, rendimiento o escalabilidad**. El atacante tipo A5 representa una amenaza poderosa, por lo que estas mitigaciones pueden no ser necesarias en muchos ecosistemas.

#### 6.3. Inyección de tokens de acceso robados

En ciertos casos, un atacante podría **inyectar un token de acceso robado** en un cliente para eludir la restricción del remitente proporcionada por [RFC8705] o [RFC9449], como se describe en el ataque "Cuckoo's Token" en [FAPI1SEC].

- **Precondiciones** para este ataque:
  1. El atacante tiene control sobre un servidor de autorización que es **confiable para el cliente**.
  2. El atacante puede comprometer la seguridad de un servidor de autorización confiable.
  3. O puede **suplantar un servidor de autorización** a través de ingeniería social.

- **Mitigaciones posibles**:
  - **Clientes** pueden usar **diferentes claves DPoP o certificados MTLS** en cada servidor de autorización.
  - Los clientes pueden enviar el **identificador del emisor** que emitió el token de acceso al servidor de recursos, y hacer que el servidor verifique que el emisor coincida con el servidor de autorización original.
  - Reducir la ventana de ataque usando **tokens de acceso de corta duración** junto con **tokens de actualización**.

#### 6.4. Fugas de solicitudes de autorización que llevan a CSRF

Un atacante de tipo A3 (ver [attackermodel]) puede interceptar una solicitud de autorización, iniciar sesión en el servidor de autorización, recibir un código de autorización y redirigir al usuario honesto mediante un ataque CSRF (Cross-Site Request Forgery) para acceder a los recursos del atacante, rompiendo así la integridad de la sesión.

- **Mitigaciones posibles**:
  1. **Requerir que el servidor de autorización acepte un `request_uri` solo una vez**, lo que impide que un atacante use la solicitud de autorización que interceptó antes de que lo haga el usuario honesto.
  2. Requerir que el cliente **realice solo una solicitud de código de autorización por cada llamada al punto final de autorización**.
  3. **Reducir la vida útil del código de autorización**, limitando la ventana de tiempo en la que se puede realizar el ataque CSRF.

#### 6.5. Ataques de intercambio de navegador

Un atacante que tenga acceso a la respuesta de autorización enviada a través del navegador de una víctima puede realizar un **ataque de intercambio de navegador**.

- **Proceso**:
  1. El atacante inicia un flujo usando su propio navegador y cliente.
  2. El cliente recibe un `request_uri` y redirige al navegador del atacante a la autorización.
  3. El atacante intercepta esta redirección y la envía a la víctima, quien podría ser engañada para autenticar y otorgar acceso.
  4. El atacante intercepta la respuesta de autorización en el navegador de la víctima, y la envía al cliente usando su propio navegador.

- **Mitigación recomendada**:
  - **Mantener la confidencialidad de la respuesta de autorización** en la redirección. La seguridad de la respuesta debe ser crítica, especialmente en contextos móviles.

#### 6.6. Implementaciones incompletas o incorrectas de las especificaciones

Es importante que la implementación de este documento y las especificaciones subyacentes de OpenID Connect y OAuth sean **completas y correctas** para lograr los beneficios de seguridad e interoperabilidad.

- **Recomendación**: Usar implementaciones certificadas por la OpenID Foundation, disponibles en:
  - [OpenID Certification Tools](https://openid.net/certification/)
  - [Lista de implementaciones certificadas](https://openid.net/developers/certified/)

#### 6.7. Suplantación del propietario del recurso por el cliente

El **cliente malicioso** podría influir en su `client_id` para que se confunda con el identificador del sujeto del usuario final. Para evitar este ataque, los servidores de autorización no deben permitir que los clientes manipulen su `client_id` de manera que pueda confundirse con un identificador de usuario final.

#### 6.8. Compromiso de claves

Si una clave criptográfica se ve comprometida, es fundamental **limitar el impacto** del evento.

- **Rotación de claves**: Se recomienda la **rotación automática y regular de claves**, para reducir la ventana de tiempo en la que una clave comprometida podría ser utilizada.
- **Alcance de la clave**: Se recomienda el uso de **claves de un solo propósito**, por ejemplo, no utilizar la misma clave para firmar y cifrar.
- **Credenciales con estado**: Si se utiliza tokens sin estado, la validación puede realizarse sin necesidad de consultas a una base de datos centralizada. Sin embargo, los tokens con estado pueden ofrecer ventajas en términos de recuperación ante la pérdida de clave.

- **Credenciales vinculadas**: Si se emiten múltiples credenciales como parte de la misma autorización, se debe establecer explícitamente su relación, para que en caso de compromiso de una, todas las credenciales puedan ser revocadas.

### 7. Consideraciones de privacidad

La implementación de este documento debe tener en cuenta muchos factores de privacidad. Dado que se trata de un perfil de OAuth 2.0 y OpenID Connect, las consideraciones de privacidad no son exclusivas de este documento, sino que generalmente se aplican a OAuth o OpenID Connect. Se recomienda a los implementadores realizar una evaluación exhaustiva de impacto en la privacidad y gestionar adecuadamente los riesgos identificados.

- **Consultas recomendadas**: Los implementadores pueden consultar documentos como [ISO29100] e [ISO29134] para este propósito.

#### Amenazas a la privacidad en implementaciones de OAuth y OpenID Connect

1. **Aviso de privacidad inapropiado**: Un aviso de privacidad (por ejemplo, proporcionado en un `policy_url`) puede ser inapropiado o insuficiente.

2. **Elección inadecuada**: Proporcionar una pantalla de consentimiento sin opciones adecuadas no constituye un consentimiento válido.

3. **Uso indebido de los datos**: Un servidor de autorización, servidor de recursos o cliente puede utilizar los datos de manera que no se haya acordado previamente.

4. **Violación del principio de minimización de la recopilación**: Si un cliente solicita más datos de los que necesita para cumplir con su propósito, está violando el principio de minimización de la recopilación.

5. **Datos personales no solicitados del servidor de recursos**: Algunas implementaciones defectuosas de servidores de recursos pueden devolver más datos de los que fueron solicitados. Si estos datos son personales, esto violaría los principios de privacidad.

6. **Violación del principio de minimización de datos**: Cualquier proceso que procese más datos de los que necesita está violando el principio de minimización de datos.

7. **Seguimiento de usuarios finales por parte de servidores de autorización**: Los servidores de autorización que identifican qué datos se están proporcionando a qué cliente y para qué usuario final.

8. **Seguimiento de usuarios finales por parte de clientes**: Dos o más clientes correlacionan tokens de acceso o ID tokens para seguir a los usuarios.

9. **Malentendido del cliente por parte del usuario final**: El usuario final no entiende correctamente quién es el cliente debido a una representación confusa del cliente en la página de autorización del servidor de autorización.

10. **Falta de comprensión por parte del usuario final al conceder acceso a los datos**: Para aumentar la confianza en el ecosistema, es una buena práctica que el servidor de autorización deje claro qué incluye la solicitud de autorización (por ejemplo, qué datos se liberarán al cliente).

11. **Ataque observando datos personales en la solicitud/respuesta de autorización**: La solicitud o respuesta de autorización podría contener datos personales. En algunas jurisdicciones, incluso los parámetros de seguridad podrían considerarse datos personales. Este perfil intenta reducir los datos enviados en la solicitud y respuesta de autorización al mínimo absoluto, pero, aun así, un atacante podría observar algunos datos.

12. **Fuga de datos del servidor de autorización**: El servidor de autorización generalmente almacena datos personales. Si se ve comprometido, estos datos pueden filtrarse o modificarse.

13. **Fuga de datos desde servidores de recursos**: Algunos servidores de recursos almacenan datos personales. Si un servidor de recursos se ve comprometido, estos datos pueden filtrarse o modificarse.

14. **Fuga de datos desde clientes**: Algunos clientes almacenan datos personales. Si el cliente se ve comprometido, estos datos pueden filtrarse o modificarse.

---

### 8. Descubrimiento de metadatos

#### 8.1. Metadatos del cliente

El siguiente metadato del cliente está definido por esta especificación:

- **use_mtls_endpoint_aliases**  
  - **Nombre del metadato**: `use_mtls_endpoint_aliases`
  - **Descripción del metadato**: Valor booleano utilizado para indicar la intención del cliente de usar TLS mutuo en lugar de los puntos finales no MTLS. Si se omite, el valor predeterminado es `false`.

---

### 9. Consideraciones de la IANA

#### 9.1. Registro de metadatos de Registro dinámico de clientes

Según esta especificación, se ha solicitado el registro de la siguiente definición de metadato del cliente en el registro **OAuth Dynamic Client Registration Metadata** de la IANA [IANA.OAuth.Parameters], establecido por [RFC7591]:

- **Nombre del metadato**: `use_mtls_endpoint_aliases`
- **Descripción del metadato**: Indica la necesidad de que un cliente utilice los alias de puntos finales de MTLS definidos por el servidor de autorización cuando estén presentes.
- **Controlador de cambios**: OIDF FAPI WG
- **Documentos de especificación**: Sección 8 del perfil de seguridad FAPI 2.0.

