# Aces — Brainstorm

> Raw ideation preserved for historical context.

> This document is not normative and should not be treated as current project specification.

# 03-09-2026
# El sueño:
Una aplicacion basada en el juego (o mas bien saga de juegos) "Advanced Wars", pero con un enfoque mas moderno, AI native, bien hecho, muy bien documentado y pensada para ser un producto/concepto/filosofia "whitelabel".

- El concepto, lo mas fundamental de "Advance Wars" (que yo recuerde), me gusta mucho y siento que puede tener muchos casos de uso. Por el momento sera solo una especie de "clon" bien hecho y moderno del juego original. Pero el concepto de equipos, usuarios, entidades con las que se puede interactuar (crear, definir un camino, ejecutar aciones), administrar recursos, gestionar eventos canonicos y sus consecuencias, dominar un entorno/ambiente/escenario con el fin de cumplir un determinado objetivo; eso es casi transversal a practicamente la vida misma.
- La idea es partir con lo chico (el juego) y darle forma a una herramienta, un producto, una filosofia que ayude a facilitar las interacciones humanas con entornos automatizados (Human-in-the-loop). Te diria que a esto me refiero con "whitelabel".

# El juego (lo que recuerdo y/o lo que me gustaria que tiviese)
- Tiene usuarios: jugadores, expectadores, moderadores, administradores. Estos ultimos los imagino agnosticos a los equipos.
- Republica: esta entidad en realidad tambien podria llamarse pais, estado, etnia, religion, proposito, grupo social, etc. Aun no lo defino, pero para efectos practicos la llamare republica.
-- La republica tiene una identidad con: nombre, gentilicio, bandera, escudo de armas, lema e himno (audio y letra), tipo (1 a 1) y categorias/tags (1 a n).
-- Una republica puede ser gestionada por 1 o mas usuarios.

# Equipos
- Determina los usuarios que comparten un mismo bando. minimo: 2, maximo: ?.
- Un equipo puede conformarse por:
-- una o varias republicas y estas a su vez podran ser gestionadas por uno o varios usuarios.
-- usuarios expectadores: no realizan acciones sobre la republica ni sus entidades. Solo ven lo mismo que puede ver un usuario jugador del mismo equipo y comunicarse con ellos (via chat puede ser, aun no lo defino)

# El mapa:
- El juego se desarrolla en un mapa compuesto por tiles hexagonales, con atributos que pueden o no ser mutables y condicionar la unidad que lo habita o transita:
- tipo de tiles:
-- agua: con niveles de corriente y profundidad.
-- tierra: con niveles de elevación y densidad de vegetación.
- una tile posee o no estructuras.
- una tile es habitada o no por una o varias unidades.
- una tile puede o no ser parte de uno o varios eventos.
- las tiles pueden o no conectarse naturalmente con sus tiles limítrofes, o mediante caminos con niveles de determinen su transitabilidad y exposicion a las unidades que lo transitan. estos caminos formaran una red de comunicaciones que le iremos dando forma mas adelante.

# Turnos por republica
- Por ahora se me ocurre que el turno respete la logica del juego original:
- Representa una ventana de acciones.
- Dado que puede haber varios jugadores por republica, para que estas acciones no se "pisen" entre jugadores, se me ocurre que solo puede haber un jugador por tile. Es decir, nunca un tile (ni la unidad/estructura que este contenga) podra ser gestionada por mas de un jugador al mismo tiempo.
- Una unidad/estructura no podra desarrollar mas de una accion por turno, salvo quiza algun evento (esto aun no lo defino bien).

# Estructuras
- Solo 1 por tile
- Tienen niveles: hasta 3 por ahora, luego definimos para que sirven y si necesitamos mas niveles.
- Tipos de estructuras:
-- base/metropolis: Representa la capital de la republica y su identidad. Si se captura por el equipo enemigo, se pierde automaticamente.
-- urbana:
--- tiene identidad: nombre, gentilicio, bandera, escudo de armas.
--- tiene niveles: caserío, población, ciudad.
--- aporta economicamente al proposito del juego.
--- tiene porcentaje de aceptación a la republica que lo posee, al equipo y/o unidad que lo transita.
-- estratégica:
--- puerto marítimo: pequeño, mediano, grande. solo tiene nombre y puede ser emplazado en la cara de un tile terrestre que limite con uno de agua.
--- aeropuerto: pequeño, mediano, grande. solo tiene nombre y puede ser emplazado en un tile terrestre plano.
--- unidad de produccion: pequeña, mediana, grande. solo tiene nombre y puede ser emplazado en un tile terrestre plano.
--- unidad de investigacion: pequeña, mediana, grande. solo tiene nombre y puede ser emplazado en un tile terrestre plano.
-- las estructuras son suceptibles a los combates que se desarrollen en su tile (y los tiles circundantes).
-- las estructuras podran o no ser suceptibles a los eventos que se desarrollen en sus tiles, a los que tengan relacion con la republica que lo represente y/o al equipo de la misma.

# Eventos
- naturales: lluvia, nieve, calor, otros.
- artificiales: generados por el usuario y/o equipo (por definir).
- se desarrollan en un numero determinado de tiles.
- pueden o no impactar a las unidades que esten en los tiles donde se desarrolla el o los eventos.
- pueden o no impactar a las estructuras que esten en los tiles donde se desarrolla el o los eventos.
- pueden impactar solo a una republica o a varias.
- pueden impactar solo a un equipo o a varios.

# la unidad:
- tiene identidad: apodo, logo/insignia, lema.
- se crea en los centros de produccion (la infanteria podra ser generada tambien en la metropolis)
- cuesta recursos
- transita entre los tiles a traves de sus limites/fronteras mediante:
- pertenece a una republica, equipo o ser neutral (npc)
- tiene atributos:
- tipos y subtipos:
-- infanteria: ligera, pesada, ataque a distancia.
-- vehiculos terrestres:
--- reconocimiento.
--- blindado: ligero, mediano, pesado.
--- artilleria: rango medio, rango largo.
--- transporte y reaprovisionamiento.
-- vehiculos acuaticos:
--- transporte.
--- patrulleras/drones.
--- submarinos.
--- fragatas (ataque antiaereo y antisubmarino)
--- destructor.
- daño por tipo: el daño que infanteria le hace a otra infanteria no es el mismo que el que puede llegar a hacer a un blindado, avion, etc.
- salud/cantidad de unidades: determina efectividad de accion, condicionada por stamina (al quedarse en cero disminuye lentamente). Porcentaje.
- rango de accion: distancia en tiles en las que una unidad puede ejecutar una accion (unidades mele: 1, unidades de rango: entre 1 y n)
- rango de movimiento por defecto: determina cuantos tiles maximos por defecto puede moverse en un turno, propia del tipo de unidad y su experiencia, condicionada por la autonomia/stamina que posea la unidad en el momento. Porcentaje.
- autonomía/stamina: determina el recurso que la unidad consume en cada turno y cuando esta se desplaza entre tiles. Porcentaje.
- experiencia: determina efectividad.
- ejecuta o recibe una accion de combate: accion ofensiva (gatillada por el usuario "dueño" de la unidad) o defensiva, gatillada por un 3ro o evento. incrementa en 1 el steps_for_next_rank de la unidad.
-- La unidad ganara experiencia mediante los "steps_for_next_rank" y existiran 3 niveles secuenciales:
--- 0: nivel con el que sale de la unidad de produccion. el steps_for_next_rank parte en cero.
--- 1: obtenido inmediatamente despues del la 3er steps_for_next_rank de forma consecutiva, o el 4to en el historico de la unidad. Al obtener este rango, el daño y la salud total (base) de la unidad incrementaran un 10% y 5% respectivamente.
--- 2: nivel maximo, inmediatamente despues del 4to steps_for_next_rank de forma consecutiva, o a la 5ta en el historico de la unidad, contadas despues de haber obtenido el rango anterior. Al obtener este rango, el daño, la salud y stamina total (base) de la unidad incrementaran un 20%, 15% y 10% respectivamente, su rango de movimiento incrementara 1 tile.

-- Las unidades del mismo tipo se pueden mergear:
--- salud y stamina: se suman
--- experiencia:

let unidad_rango_superior
let unidad_rango_inferior
let nueva_unidad_fusionada
let diferencia_porcentual_entre_saludes # Diferencia porcentual estándar (Fórmula del punto medio) entre la salud de ambas entidades a mergear.

if unidad_rango_superior.salud >= unidad_rango_inferior.salud
    nueva_unidad_fusionada.exp = unidad_rango_suoperior.exp
elsif unidad_rango_superior.salud >= 40%
    if  diferencia_porcentual_entre_saludes <= 50%
        nueva_unidad_fusionada.exp = unidad_rango_suoperior.exp
    else
        nueva_unidad_fusionada.steps_for_next_rank++
    end
else # unidad_rango_superior.salud < unidad_rango_inferior.salud
    nueva_unidad_fusionada.steps_for_next_rank++
end

* Este formula esta sujeta a cambios y sugerencias.

* Como ya es costumbre, me gustaria que primero investigues los conceptos, fundamentos, convenciones y antecedentes detras de estas ideas; para darle forma a un proyecto bien hecho, bien documentado, AI native y con proposito.

* Considera que este seria mi primer approach a los videojuegos, asi que hay mucho de aca que me gustaria que iteraramos juntos. Sientete libre de sugerir cualquier cambio.