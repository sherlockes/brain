+++
title = "jinja"
author = ["Sherlockes"]
date = 2024-01-14
lastmod = 2024-06-05T18:20:12+02:00
tags = ["jinja"]
categories = ["script"]
draft = false
weight = 5
thumbnail = "images/jinja.png"
toc = true
+++

Jinja2 es un motor de plantillas para el lenguaje de programación Python. Fue creado por Armin Ronacher y se utiliza comúnmente en aplicaciones web basadas en Python para generar contenido dinámico. Jinja2 permite la creación de plantillas que contienen marcadores de posición y lógica de control, que luego pueden ser llenados con datos específicos cuando se renderiza la plantilla.

<!--more-->

Estos son los conocimientos que he ido adquiriendo sobre jinja 2 para Home Assistant.

Mi uso se restringe básicamente a la generación de plantillas para [Home Assistant]({{< relref "iot.md" >}}). Puedes ver como lo he utilizado en la generación de [sensores](https://github.com/sherlockes/ha_cfg/blob/main/sensors.yaml) y [sensores binarios](https://github.com/sherlockes/ha_cfg/blob/main/binary_sensors.yaml) virtuales.


## Entidades {#entidades}

-   Obtener las entidades de una integración `{{ integration_entities('<integración>') }}`
-   Obtener el atributo de una entidad `{{ state_attr('<entidad>', '<atributo>') }}`
-   Obtener el estado de una entidad `states('<entidad>')`

---


## Bucles {#bucles}

-   If
    ```jinja2
    {% if <condicion_1> %}
    Resultado 1
    {% elif <condicion_2> %}
    Resultado 2
    {% else %}
    Resultado 3
    {% endif %}
    ```
-   If not
    ```jinja2
    {% if not <condicion_1> %}
    Resultado 1
    {% endif %}
    ```
-   For
    ```jinja2
    {% for <elemento> in <rango> %}
      ...
    {% endfor %}
    ```

---


## Listas {#listas}

```jinja
{% lista = namespace(numbers=[]) %}
{% set lista.numbers = precios.numbers + [precio] %}
```


## Manejo de horas y fechas {#manejo-de-horas-y-fechas}

-   Ahora `{{ now() }}`
-   Ahora (solo fecha) `{{ now().date() }}`
-   Ahora (solo hora) `{{ now().hour }}`
-   Convertir a segundos `{{ as_timestamp(<fecha>) }}`
-   Incrementar un día `{{ (now() + timedelta(days=1)) }}`
-   Día de la semana `{{ (<fecha>).strftime('%w') }}`
-   Día de la semana `{{ as_timestamp(<fecha>) | timestamp_custom('%w') }}`
-   De lunes a viernes `{{ (<fecha>).strftime('%w') | int in range(1, 6) }}`
-   Despreciar segundos `{{ now().replace(hour=8, minute=0, second=0, microsecond=0) }}`
-   Texto a fecha `as_datetime('YYYY-MM-DDTHH:MM:SS')`
-   Reemplazar `{{ as_datetime('2023-05-21').replace(hour=8, minute=0, second=0, microsecond=0) }}`

---


## Operaciones {#operaciones}

-   length -&gt; Elementos de un rango o lista
-   int -&gt; Convierte a entero
-   float -&gt; Convierte a decimal
-   max -&gt; devuelve el valor máximo de una lista
-   min -&gt; devuelve el valor mínimo de una lista

---


## Rangos {#rangos}

-   Desde hasta `{{ <number> | int in range(1, 6) }}`
-   Valores `{{ <number> | int in ["1", "2", "6") }}`

---


## Ejemplos {#ejemplos}


### Incremento de tiempo {#incremento-de-tiempo}

Añade al ayudante "input_datetime.tv_off_kids" 60 segundos

```jinja2
{{(((state_attr('input_datetime.tv_off_kids' , 'timestamp')) + 60) |
    timestamp_custom('%H:%M', false))}}
```

> Si en la función "timestamp_custom" ponemos "true" como segundo parámetro tenemos que restar 3600 sg (1 hora) a los 60 segundos para obtener el resultado adecuado. Para conseguir la hora local hay que poner este argumento a "false".

Si queremos restar un número de minutos definido por el ayudante "counter.malos_habitos" modificaremos la función anterior quedando lo siguiente.

```jinja2
{{(((state_attr('input_datetime.tv_off_kids' , 'timestamp')) - (60 * ((states('counter.malos_habitos') | float)))) |
  timestamp_custom('%H:%M', false))}}
```

---


### Encendido de la calefacción entre dos horas {#encendido-de-la-calefacción-entre-dos-horas}

Tenemos dos ayudantes, (hora de encender y hora de apagar) entre las cuales el resultado de la función deberá ser "on", "off" en el resto de horario"

```jinja2
{% set hora_actual = now().strftime('%H:%M') %}
{% set hora_encendido = states('input_datetime.calefaccion_hora_de_encender') %}
{% set hora_apagado = states('input_datetime.calefaccion_hora_de_apagar') %}
{% set temperatura_exterior = states('sensor.temperatura_exterior') | float %}
{{ 'on' if hora_encendido <= hora_actual < hora_apagado and temperatura_exterior < 15 else 'off' }}
```

---


### Determinar si mañana hay cole {#determinar-si-mañana-hay-cole}

-   Hay un calendario (calendar.festivos_escolares) con los días festivos
-   Hay un ayudante (input_boolean.hay_cole_manana) que almacena el valor
-   Hay un script (Cole mañana) que recoge los eventos del calendario de las próximas 24 horas y lo guarda en la variable (agenda_calendario)

En un primer momento parece obvio que si mañana es sábado o domingo no habrá cole

Con esto necesitamos una plantilla para que, a partir de "calendar.festivos_escolares" devuelva "true" si mañana hay cole o "false" si no lo hay. Habrá cole si:

-   No hay ningún evento en el calendario
-   El evento que hay termina antes de mañana (Hoy no es lectivo pero mañana si)

<!--listend-->

```jinja2
{# Define tomorrow como la fecha de mañana a estas horas #}
{% set tomorrow = as_timestamp((now() + timedelta(days=1)).date()) %}

{# Si mañana no es de lunes a viernes, false, no hay cole #}
{% if not (now() + timedelta(days=1)).strftime('%w') | int in range(1, 6) %}
  false
{# Si mañana no hay ningún evento en "festivos_escolares", true, hay cole #}
{% elif not agenda_calendario["calendar.festivos_escolares"]["events"] %}
  true
{% else %}
  {% for event in agenda_calendario["calendar.festivos_escolares"]["events"] %}
    {% set event_end = as_timestamp(strptime(event.end, "%Y-%m-%d")) %}
    {# Si para mañana se ha terminado el evento, true, hay cole #}
    {% if tomorrow > event_end %}
      true
      {% break %}
    {% else %}
      false
    {% endif %}
  {% endfor %}
{% endif %}
```

---


### Las persianas de casa {#las-persianas-de-casa}


#### Bajar las persianas cuando levanta el sol y hace calor {#bajar-las-persianas-cuando-levanta-el-sol-y-hace-calor}

Hay que determinar cuando está levantando el sol y hace el suficiente calor dentro o fuera de casa

-   Elevación del sol entre 21º y 50º
-   La hora menor de las 14:00 para despreciar cuando se esconde el sol
-   Temperatura interior o exterior por encima de 23 ºC

<!--listend-->

```jinja2
{% set elev = 21 < state_attr('sun.sun', 'elevation') < 50 %}
{% set hora = now().hour < 14 %}
{% set exterior_temperature = states('sensor.temperatura_exterior') | float %}
{% set salon_temperature = states('sensor.temperatura_salon') | float %}
{% set temp = exterior_temperature > 23 or salon_temperature > 23 %}
{{ elev and hora and temp }}
```


#### Bajar las persianas cuando hace frío y se esconde el sol {#bajar-las-persianas-cuando-hace-frío-y-se-esconde-el-sol}

```jinja2
{% set ta_minima = states('sensor.aemet_daily_forecast_temperature_low') | float %}
{% set sun_elevation = state_attr('sun.sun', 'elevation') %}
{% set exterior_temperature = states('sensor.temperatura_exterior') | float %}
{{ sun_elevation < -5 and now().hour < 22 and (ta_minima < 10 or exterior_temperature < 14) }}
```

---


#### Cuando el sol está entrando por una ventana {#cuando-el-sol-está-entrando-por-una-ventana}

En función de la altura del sol, el acimut, la consigna de temperatura de la caldera  y la temperatura exterior, esta plantilla devuelve verdadero cuando el sol está entrando por una ventana.

```jinja
{% set cal = state_attr('climate.caldera', 'temperature' ) + 4 %}
{% set temp = states('sensor.temperatura_exterior')|int %}
{% set ele = state_attr('sun.sun', 'elevation') %}
{% set azi = state_attr('sun.sun', 'azimuth') %}
{{ temp > cal and azi > 230 and ele > 45 }}
```

---


### Dispositivos Zigbee no disponibles {#dispositivos-zigbee-no-disponibles}

Es importante saber cuando algún dispositivo Zigbee no está disponible, para ello utilizo el siguiente script

```jinja2
{% set zha_entities = integration_entities('zha') %}
{% set mqtt_entities = integration_entities('mqtt') %}
{% set zigbee_entities = zha_entities + mqtt_entities %}
{% set zigbee = namespace(unavailable=[]) %}

{% for entity in zigbee_entities %}
  {% if states(entity) == 'unavailable' %}
    {% set nombre = device_attr(device_id(entity), 'name_by_user') %}
    {% if nombre not in zigbee.unavailable %}
      {% set zigbee.unavailable = zigbee.unavailable + [nombre] %}
    {% endif %}
  {% endif %}
{% endfor %}
{{ zigbee.unavailable | length }}
```

Tengo activas dos integraciones con las que controlo dispositivos zigbee, "zha" y "mqtt" por lo que habrá que buscar entidades no disponibles en ambas.

---


### Calcular el ahorro en la factura de la luz según la hora {#calcular-el-ahorro-en-la-factura-de-la-luz-según-la-hora}

Partimos de la integración "Spain electricity hourly pricing (PVPC)" que nos genera la entidad "sensor.esios_pvpc" que entre otros tiene los siguientes atributos:

-   price_00h: 0.06117
-   price_01h: 0.05735
-   price_02h: 0.05798
-   price_03h: 0.05663
-   price_04h: 0.05656
-   ...

A partir de aquí generaremos una lista con todos los precios según la hora del día y calcularemos el ahorro porcentual de la hora actual en función del precio máximo y mínimo de la energía en el día.

```jinja
{% set precios = namespace(numbers=[]) %}
{% for hora in range(0, 24) %}
  {% set key = 'price_' ~ '{:02d}'.format(hora) ~ 'h' %}
  {% set precio = state_attr ('sensor.esios_pvpc', key) %}
  {% set precios.numbers = precios.numbers + [precio] %}
{% endfor %}

{% set max_ahorro = precios.numbers | max - precios.numbers | min %}
{% set now_ahorro = precios.numbers | max - precios.numbers[now().hour] %}
{% set ahorro = ((now_ahorro * 100)/max_ahorro) | int%}

{{ ahorro }}
```

---
