+++
title = "BauEngine"
date = "2026-11-15"
description = "Programando un motor de Visual Novels para Monogame"
thumbnail = "/images/noimage.jpg"
tags = [ "Development", "Visual Novel" ]
+++

Mi último año en temas de desarrollo ha sido digamos que curioso.

Llevaba ya algunos meses buscando algo que programar en mi tiempo libre que tuviera poco o nada que ver con mi trabajo y
comencé a interesarme en la programación de **Visual Novel**, como no podía ser de otra forma, más en los motores
de programación de **Visual Novel** como [RenPy](https://www.renpy.org/).

Mi primera intención era hacer algo sencillo: definir un metalenguage de visual novel con XML y crear un motor para
presentación de imágenes y textos con WPF. Algo similar a lo que hice con [BauMotionComics](https://github.com/jbautistam/BauMotionComics)
hace casi diez años pero mejor.

La idea no estaba mal, lo malo es que cuanto más opciones añadía al lenguage, como por ejemplo transiciones o animaciones, más complicado
era manejarlo con WPF. No era imposible, pero sí que me daba cuenta que no era la elección más adecuada para ello.

Y recordé [MonoGame](https://monogame.net/).

Para quien no lo conozca, **Monogame** es un framework de creación de videojuegos que nace de XNA (el SDK de Microsoft para desarrollo
en XBox). Le había perdido de pista desde hacía unos años aunque ya había hecho algunos experimentos como
[CrioGame](/blog/articles/source-code/CrioGame/criogame).

En estos años ha seguido evolucionando y ha adquirido algo más de fama. Sobre el framework se han escrito videojuegos como 
[Stardew Valley](https://store.steampowered.com/app/413150/Stardew_Valley) o [Celeste](https://www.celestegame.com/) y comienza
a tener su público.

**Monogame** es un framework, no un motor de videojuegos como Unreal, Unity o Godot. Esto significa que ofrece
métodos para dibujar gráficos o audio y control de tiempos para los métodos típicos Update / Draw pero poco más.

Es decir, si quieres crear una animación, la respuesta de **Monogame** es "búscate la vida", si quieres dibujar un botón y recibir
las pulsaciones, la respuesta de **Monogame** es "búscate la vida", si quieres gestionar una cámara, la respuesta de **Monogame**
es "búscate la vida"... suficiente, ya nos entendemos.

Si no quieres volverte demasiado loco puedes utilizar una de las librerías asociadas a **Monogame** como [Monogame Extended](https://www.monogameextended.net/),
si eres como yo y te gustan los retos, puedes crearte algo similar a un motor de videojuegos y sobre él comenzar a preparar tu
motor de programación de **Visual Novel** que era para lo que estábamos aquí.

Así que lo que tengo ahora mismo es un motor simple de videojuegos con tratamiento de escenas, capas, animaciones, cámara, componentes de
interface de usuario, físicas y actores y sobre él he construido un sistema para mostrar Visual Novel.

Este es el primer capítulo de ejemplo que estoy utilizando mientras desarrollo (que no va a ganar ningún premio como Visual Novel 
pero me permite ir jugando): [YouTube](https://www.youtube.com/watch?v=YsjElg4n2eM).

<iframe width="1331" height="826" src="https://www.youtube.com/embed/YsjElg4n2eM" title="Pruebas de motor de Visual Novel - 01" frameborder="0" 
	allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" 
	referrerpolicy="strict-origin-when-cross-origin" allowfullscreen>
</iframe>

Que, por si tenéis curiosidad, se interpreta a partir de este script:

```
#import "VisualNovel/Common/sequences-definition.scn"

/* Asocia eventos a secuencias */
treatEvent "start-say" "start-say-sequence"
treatEvent "end-say" "end-say-sequence"

/* Carga la pantalla con su música y el fondo */ 
screen "VisualNovel/Screens/VisualNovelUiScreen.xml"
play "sounds/music/mihailandrei-caminho-de-luz"
show "bg-home"

/* Introducción */
say "narrator" "El lema oficial de la Universidad Luminae reza: 'Donde las estrellas guían tu destino'."
say "narrator" "Lo que no mencionan en el folleto es que las estrellas también se equivocan."
say "narrator" "Y a veces, literalmente, explotan en confeti."

/* Fade in de Elara */
sequence "fade-in" [ "wait": true ]
{
    track "elara"
    {
        step "Reset" [ "x": 0.3, "y": 1.3, "opacity": 0, "rotation": 0 ]
        step "Fade" [ "opacity": 1, "duration": 0.3 ]
    }
}

say "elara" "Sólo llevo tres minutos mirando este letrero y mi maleta ya está intentando abrirse sola"
say "elara" "Ya que estamos aquí, habrá que aprovechar el tiempo y entrar en la escuela"
say "elara" "Semestre uno: sobrevivir. Semestre dos: tal vez descansar un poco"

hide "dialog"

/* Fade in de nuevo fondo, ocultar Elara y fondo antiguo */
sequence "fade-in-background" [ "wait": true ]
{
    track "bg-square"
    {
        step "Reset" [ "x": 0, "y": 0, "opacity": 0, "rotation": 0 ]
        step "Fade" [ "opacity": 1, "duration": 0.3 ]
    }
    track "elara"
    {
        step "Fade" [ "opacity": 0, "duration": 0.3 ]
    }
}

hide "bg-home"
hide "elara"

/* Presenta los personajes */
show "james/happy/1" [ "x": -1, "y": 1.2, "opacity": 1, "rotation": 0, "lookAt": "RightToLeft" ]
show "sylvie/happy/1" [ "x": 0.5, "y": 1.2, "opacity": 0, "rotation": 0, ]
show "mark/shy/1" [ "x": 2, "y": 1.2, "opacity": 1, "rotation": 0, "lookAt": "LeftToRight"  ]
sequence "presentation" [ "wait": true ]
{
    track "james"
    {
        step "Translate" [ "targetX": 0.2, "targetY": 1.2, "start": 0.01, "duration": 1 ]
    }
    track "sylvie"
    {
        step "fade" [ "opacity": 1, "duration": 2 ]
    }
    track "mark"
    {
        step "Translate" [ "targetX": 0.9, "targetY": 1.2, "start": 0.01, "duration": 1 ]
    }
}

say "james" "¡Ciudadanos del cosmos! La ceremonia de asignación estelar comienza en exactamente tres minutos y medio"
say "james" "Por favor, alineen sus varas, ajusten sus diademas y dejen de alimentar a las gárgolas."

say "sylvie" "Respira, James. Y recuerda: si sientes opresión en el pecho, toma tres sorbos de té de lavanda antes de desmayarte. El kit está en mi mochila, tercera bolsa."

say "mark" "¿O qué? ¿Que las estrellas nos juzguen? Francamente, el universo ni siquiera nos responde los mensajes. Y aún así, aquí estamos."

avatar "elara" [ "position": "left", "visible": true ]
say "elara/thinking" "Vale. Tres personalidades. Tres ritmos distintos. Y yo en medio intentando que mi maleta no huya."
avatar "elara" [ "position": "left", "visible": false ]

hide "dialog"

choice "¿Qué quieres hacer ahora?" [ "DialogTemplate": "SelectorNormal", "OptionTemplate": "SelectorOptionNormal", 
                                     "position": "0.3, 0.3, 0.3, 0.4" ]
{
    case "Reaccionar con entusiasmo ordenado" [ "texture": "james-character", "region": "avatar-shy" ]
    {
        set tono_principal = "pragmatico"
        say "elara" "Tres minutos y medio. Suficiente para revisar el protocolo de emergencia y ubicar la salida más cercana."
        show "james/happy/2"
        say "james" "¡Exacto! Disciplina. Me gusta tu espíritu, Elara. La academia necesita mentes claras."
    }
    case "Ofrecer apoyo empático" [ "texture": "sylvie-character", "region": "avatar-shy" ]
    {
        set tono_principal = "empatico"
        say "elara" "Están bastante tensos. ¿Alguna vez han probado hacer una pausa para respirar juntos? El estrés no debería ser parte del uniforme."
        show "sylvie/happy/2"
        say "sylvie" "Oh, gracias. En serio. A veces olvido que el autocuidado también es colectivo. Toma, ¿quieres un caramelo de menta estelar?"
    }
    case "Hacer un comentario absurdo" [ "texture": "mark-character", "region": "avatar-shy" ]
    {
        set tono_principal = "caotico"
        say "elara" "Si las estrellas se equivocan, ¿alguien ha pensado en pedirles reembolso? O al menos un vale por un buen café que no sepa a polvo de meteorito."
        show "mark/surprised/1"
        say "mark" "Finalmente, alguien que habla mi idioma. El cosmos es un chiste mal contado, pero al menos tiene snacks. Bienvenida al club de los desorientados, Elara."
    }
}

/* Mueve los personajes */
show "rector" [ "x": 2, "y": 1.2 ]
sequence "fade-in-background" [ "wait": true ]
{
    track "james"
    {
        step "Translate" [ "targetX": 0.1, "targetY": 1.2, "start": 0.01, "duration": 1 ]
    }
    track "sylvie"
    {
        step "Translate" [ "targetX": 0.2, "targetY": 1.2, "start": 0.01, "duration": 1 ]
    }
    track "mark"
    {
        step "Translate" [ "targetX": 0.3, "targetY": 1.2, "start": 0.01, "duration": 1 ]
    }
    track "rector"
    {
        step "Translate" [ "targetX": 0.7, "targetY": 1.2, "start": 0.01, "duration": 1 ]
    }
}

say "narrator" "El círculo de invocación estelar parpadea. En lugar de un símbolo limpio, aparece... un garabato."
say "narrator" "No. Un garabato con opiniones."
    
show "rector/scared/1"
say "rector" "¡Por las esferas giratorias! ¡Es una Constelación Fracturada! ¡Nadie ha visto una en siglos!"

show "james" [ "layer": 25 ]
say "james/scared/1" "¿Eso es... peligroso?"
show "sylvie" [ "layer": 30 ]
say "sylvie/scared/1" "O increíblemente estresante para el sistema nervioso."
show "mark" [ "layer": 35 ]
say "mark/scared/2" "O ambas. Mi gato lleva un rato escondido bajo el sofá.[wait=2] Y ni siquiera tengo gato."
    
say "elara" "Tranquilos, no pasa nada. Seguro que es solo... ¿magia de temporada?"

show "rector/angry/1"
say "rector" "¡Nada de 'solo'! Si no se estabiliza antes del Festival de Cierre, el Núcleo Celestial colapsará."
show "rector/angry/2"
say "rector" "Y el campus se convertirá en una tormenta de confeti, café derramado y pergaminos que insultan a sus autores."
    
say "elara" "¿Y cómo se supone que podemos arreglarlo?"

show "rector/angry/4"
say "rector" "Reuniendo Anclajes de Resonancia. Tres puntos del campus. ¡Y por las estrellas, no toques nada brillante sin guantes!"
    
/* Mueve los personajes */
sequence "fade-out-characters" [ "wait": true ]
{
    track "james"
    {
        step "fade" [ "opacity": 0, "duration": 1 ]
    }
    track "sylvie"
    {
        step "fade" [ "opacity": 0, "duration": 1 ]
    }
    track "mark"
    {
        step "fade" [ "opacity": 0, "duration": 1 ]
    }
    track "rector"
    {
        step "fade" [ "opacity": 0, "duration": 1 ]
    }
}
hide "james"
hide "sylvie"
hide "mark"
hide "rector"

avatar "elara/angry" [ "position": "left", "visible": true ]
say "elara" "Bien, Elara. Paso uno, sobrevivir a la orientación, hecho."
avatar "elara/angry" [ "position": "left", "visible": false ]

show "elara/angry/2" [ "x": 0.5, "y": 1, "opacity": 1 ]
say "elara" "Paso dos: encontrar esos anclajes antes de que el rector me convierta en nota al pie de página."
    
if (tono_principal == "empatico")
{
    say "narrator" "Sientes un peso suave en el pecho. No es miedo. Es anticipación. El aire huele a tinta vieja y a promesas a medio cumplir."
}

hide "dialog"

say "narrator" "Constelación registrada. 

                 Semestre iniciado. 
                 
                 Que el caos te acompañe [color=green](pero con moderación)[/color]."
        [ "template": "DialogFullScreen" ]

/* Graba y salta al siguiente capítulo */
save "Chapter 2" "VisualNovel/chapters/chapter-02.scn" [ "texture": "thumbs", "region": "th-livingroom" ]
jump "VisualNovel/chapters/chapter-02.scn"
```

Por supuesto, sigue siendo un desarrollo en curso y probablemente llevará otro año más para que esté completamente satisfecho con él, pero ya va
cogiendo forma y muchas de las características que quería como animaciones o secuencias cinemáticas, ya están implementadas así que os mantendré
informados.