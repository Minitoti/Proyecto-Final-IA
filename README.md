# Proyecto-Final-IA
* Marly Ceballes Arias
* Samuel Concha

## Objetivo
Este código se trata de un proyecto final de IA que utiliza algoritmos genéticos para obtener el camino mínimo en un laberinto de tamaño X por X. A continuación, se explica el código por secciones:

### Librerias
**numpy:** se usa para generar matrices de tamaño NxN  llamadas mt y vis para simular los recorridos y bloqueos que se van a generar en el mapa.
**random:**  permite generar números aleatorios y se usa para escoger puntos de bloqueos aleatorios. 
**pygame:** Es una biblioteca de programación de juegos en Python.
**os:** Es una biblioteca que proporciona funciones para interactuar con el sistema operativo. Las funciones **listdir** y **isfile** se utilizan para obtener una lista de archivos en un directorio y verificar si una ruta dada corresponde a un archivo existente, respectivamente. Estas funciones son útiles para trabajar con archivos y directorios en Python.
**solucionadorLaberinto:** Es un módulo personalizado que contiene funciones específicas para resolver laberintos.
**time:** Esta biblioteca proporciona funciones relacionadas con el tiempo.
```
import pygame
import random
import numpy as np
from os import listdir
from os.path import isfile, join
import solucionadorLaberinto as solved
import time
```
### Colores
Se definen constantes para representar colores en formato RGB, como el color negro (BLACK) y el color blanco (WHITE).

```
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
```
### Dimensiones de la ventana
Se definen las dimensiones de la ventana del juego mediante las variables **WINDOW_WIDTH** y **WINDOW_HEIGHT**.

```
WINDOW_WIDTH = 800
WINDOW_HEIGHT = 600
```

### Inicialización de Pygame
Se inicializa la biblioteca Pygame y se crea una ventana de juego con las dimensiones especificadas anteriormente. También se establece el título de la ventana.

```
pygame.init()
menu_window = pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
pygame.display.set_caption("Menú de Juego")
```

### Fuente
Se crea un objeto de fuente utilizando la fuente "Arial" con un tamaño de 32 píxeles. Esta fuente se utilizará más adelante para renderizar texto en la ventana.

```
font = pygame.font.SysFont("Arial", 32)
```

### Variables de juego
Se definen varias variables que se utilizarán para el funcionamiento del juego. Estas variables incluyen la dificultad seleccionada, un indicador de si el juego ha comenzado y una clase llamada **Menu** que se utiliza para crear y mostrar el menú del juego.

```
selected_difficulty = 0  // Dificultad seleccionada (Facil = 0, Medio = 1, Dificil = 2)
game_started = False  // Indica si el juego ha comenzado
```

### Clase Menu
Esta clase representa el menú del juego y tiene métodos para dibujar el menú en la ventana.
El constructor de la clase recibe la ventana del juego y la ruta de la imagen de fondo del menú.
El método draw_menu dibuja los elementos del menú, como el título, las opciones de dificultad y el botón de inicio.

```
class Menu:
    def __init__(self, ventana, pathFondo):
        self.ventana = ventana
        self.posicionBotonJugar = (WINDOW_WIDTH // 2, 500)
        self.rectBotonJugar = pygame.Rect(0,0, 0, 0)
        self.rectDificultad = [pygame.Rect(0,0,0,0)] * 3
        self.fondo = pygame.image.load(pathFondo)
        self.fondo = pygame.transform.scale(self.fondo, (WINDOW_WIDTH, WINDOW_HEIGHT))

    def draw_menu(self):
        self.ventana.blit(self.fondo, (0,0))
        // Título
        title_text = font.render("Menú de Juego", True, WHITE)
        title_rect = title_text.get_rect(center=(WINDOW_WIDTH // 2, 100))
        self.ventana.blit(title_text, title_rect)
```

### Opciones de dificultad
En esta sección del código, se definen las opciones de dificultad para el juego. Estas opciones permiten ajustar la dificultad del laberinto, lo cual afectará la cantidad de obstáculos y la complejidad del mismo. 

```
        dificultad_text = font.render("Seleccionar dificultad", True, WHITE)
        dificultad_rect = dificultad_text.get_rect(center=(WINDOW_WIDTH // 2, 150))
        self.ventana.blit(dificultad_text, dificultad_rect)
        difficulties = ["Facil: 10x10", "Medio: 20x20", "Dificil: 30x30"]
        colores = [[(25, 111, 61), (183, 149, 11), (148, 49, 38)],
                [(169, 223, 191), (249, 231, 159 ), (250, 219, 216)]]
        for i, difficulty in enumerate(difficulties):
            text = font.render(difficulty, True, WHITE)
            self.rectDificultad[i] = text.get_rect(center=(WINDOW_WIDTH // 2, 200 + i * 50))
            if i == selected_difficulty:
                text = font.render(difficulty, True, colores[0][i])
                # self.rectDificultad[i] = text.get_rect(center=(WINDOW_WIDTH // 2, 200 + i * 50))
                pygame.draw.rect(self.ventana, colores[0][i], self.rectDificultad[i].inflate(10, 10), 5)
                pygame.draw.rect(self.ventana, colores[1][i], self.rectDificultad[i])
            self.ventana.blit(text, self.rectDificultad[i])
        # Botón de inicio
        start_text = font.render("Iniciar Juego", True, WHITE)
        self.rectBotonJugar = start_text.get_rect(center=(self.posicionBotonJugar))
        pygame.draw.rect(self.ventana, WHITE,  self.rectBotonJugar.inflate(10, 10), 2)
        self.ventana.blit(start_text, self.rectBotonJugar)

def start_game():
    global game_started
    game_started = True

menu = Menu(menu_window, "assets/artwork.png")
````

### Ventana de juego
Se definen algunas constantes relacionadas con la ventana de juego, como el ancho, alto y FPS.
* Se define la función obtener_fondo() para cargar y generar una lista de posiciones de tiles de fondo.
* Se define la función get_block() para cargar y generar una imagen de bloque.
* Se define la función flip() para invertir sprites en dirección horizontal.
* Se define la función load_fruit() para cargar y generar sprites de frutas.
* Se define la función load_sprite_sheets() para cargar y generar sprites de personajes.
  
```
WIDTH , HEIGHT = 800 , 600
FPS = 60
PLAYER_VEL = 4

// window = pygame.display.set_mode((WIDTH , HEIGHT))

def obtener_fondo(name):
    image = pygame.image.load(join("assets","Background",name)) # Carga la imagen
    _, _, width, height = image.get_rect() # da el tamaño de la imagen y las posiciones, en este caso no importan 
    tiles=[]
    
    for i in range(WIDTH//width + 1):
        for j in range(HEIGHT//height + 1):
            pos =(i * width, j * height)
            tiles.append(pos)
            
    return tiles, image

def get_block(size):
    path = join("assets","Terrain","Terrain.png")
    image = pygame.image.load(path).convert_alpha()
    surface = pygame.Surface((size,size) , pygame.SRCALPHA , 32)
    rect = pygame.Rect(8*26, 16 , size , size)
    surface.blit(image , (0,0) , rect)
    return pygame.transform.scale2x(surface)

def flip(sprites):
    return[pygame.transform.flip(sprite,True,False)for sprite in sprites]

def load_fruit(name,width,height):
    path=join("assets","Items",name)
    images=[f for f in listdir(path) if isfile(join(path , f))]
    
    all_sprites = {}
    
    for image in images:
        sprite_sheet=pygame.image.load(join(path,image)).convert_alpha()
        
        sprites=[]
        for i in range(sprite_sheet.get_width()//width):
            surface = pygame.Surface((width,height), pygame.SRCALPHA, 32)
            rect=pygame.Rect(i*width, 0, width, height)
            surface.blit(sprite_sheet, (0,0), rect)
            sprites.append(pygame.transform.scale2x(surface))
        all_sprites[image.replace(".png","")] = sprites
            
    return all_sprites

def load_sprite_sheets(dir1, dir2, width, height, direction=False):
    path = join("assets",dir1, dir2)
    images=[f for f in listdir(path) if isfile(join(path , f))]
    
    all_sprites = {}
    
    for image in images:
        sprite_sheet=pygame.image.load(join(path,image)).convert_alpha()
        
        sprites=[]
        for i in range(sprite_sheet.get_width()//width):
            surface = pygame.Surface((width,height), pygame.SRCALPHA, 32)
            rect=pygame.Rect(i*width, 0, width, height)
            surface.blit(sprite_sheet, (0,0), rect)
            sprites.append(pygame.transform.scale(surface,(40,40)))
            
        if direction:
            all_sprites[image.replace(".png","")+ "_right"]= sprites
            all_sprites[image.replace(".png","")+ "_left"] = flip(sprites)
        else:
            all_sprites[image.replace(".png","")] = sprites

    return all_sprites
```
### Clase Jugador 
Esta clase hereda de la clase pygame.sprite.Sprite y representa al jugador en el juego. Tiene métodos para mover al jugador en diferentes direcciones y actualizar su sprite en función de su movimiento. También tiene un método draw para dibujar al jugador en la ventana del juego.

```
class Jugador(pygame.sprite.Sprite):
    
    SPRITES=load_sprite_sheets("MainCharacters","MaskDude",32,32,True)
    ANIMATION_DELAY = 3
    
    def __init__(self,x,y,width,height):
        super().__init__()
        self.rect = pygame.Rect(x,y,width,height)
        self.x_vel = 0
        self.y_vel = 0
        self.mask = None
        self.direction = "left"
        self.animation_count = 0
        
    def move(self,dx,dy):
        self.rect.x += dx
        self.rect.y += dy
        
    def move_left(self,vel):
        self.x_vel = -vel
        if self.direction != "left":
                self.direction = "left"
                self.animation_count = 0
    def move_right(self,vel):
        self.x_vel = vel
        if self.direction != "right":
                self.direction = "right"
                self.animation_count = 0
    def move_up(self,vel):
        self.y_vel = -vel
        self.animation_count = 0
        
    def move_down(self,vel):
        self.y_vel = vel
        self.animation_count = 0
        
    def loop(self,FPS):
        self.move(self.x_vel,self.y_vel)
        self.update_sprite()
        self.update()
        
    def update_sprite(self):
        sprite_sheet= "idle"
        if self.y_vel != 0 :
            sprite_sheet = "run"
        elif self.x_vel != 0:
            sprite_sheet = "run"
        
        sprite_sheet_name = sprite_sheet + "_" + self.direction
        sprites = self.SPRITES[sprite_sheet_name]
        sprite_index = (self.animation_count // self.ANIMATION_DELAY) % len(sprites)
        self.sprite = sprites[sprite_index]
        self.animation_count += 1
        self.update()
        
    def update(self):
        self.rect = self.sprite.get_rect(topleft=(self.rect.x,self.rect.y))
        self.mask = pygame.mask.from_surface(self.sprite)
        
    def draw(self,win, offsetx,offsety):
        win.blit(self.sprite, (self.rect.x - offsetx, self.rect.y - offsety))
```
### Clase Object
Esta clase también hereda de pygame.sprite.Sprite y representa un objeto en el juego. Tiene un método draw para dibujar el objeto en la ventana.

```
class Object(pygame.sprite.Sprite):
    def __init__(self , x , y , width , height , name=None):
        super().__init__()
        self.rect=pygame.Rect(x , y , width , height)
        self.image = pygame.Surface((width , height), pygame.SRCALPHA)
        self.width = width
        self.height = height
        self.name = name

    def draw(self , win, offsetx,offsety):
        win.blit(self.image , (self.rect.x - offsetx, self.rect.y - offsety))
```

### Clase Fruit
Esta clase hereda de la clase Object y representa una fruta en el juego.
Tiene un método loop que se llama en cada iteración del bucle principal del juego para actualizar la posición y el sprite de la fruta.

```
class Fruit(pygame.sprite.Sprite):
    SPRITES=load_fruit("Fruits",32,25)
    ANIMATION_DELAY = 3
    Frutas=random.choice(["Apple","Bananas","Cherries","Kiwi","Melon","Orange","Strawberry"])
    
    def __init__(self,x,y,width,height):
        super().__init__()
        self.rect = pygame.Rect(x,y,width,height)
        self.x_vel = 0
        self.y_vel = 0
        self.mask = None
        self.animation_count = 0
        self.collide=0
        
    def move(self,dx,dy):
        self.rect.x += dx
        self.rect.y += dy
        
    def loop(self,FPS):
        self.move(self.x_vel,self.y_vel)
        self.update_sprite()
        self.update()
        
    def update_sprite(self):
        sprite_sheet= self.Frutas 
        sprite_sheet_name = sprite_sheet
        sprites = self.SPRITES[sprite_sheet_name]
        sprite_index = (self.animation_count // self.ANIMATION_DELAY) % len(sprites)
        self.sprite = sprites[sprite_index]
        self.animation_count += 1
        self.update()
   
    def update(self):
        self.rect = self.sprite.get_rect(topleft=(self.rect.x,self.rect.y))
        self.mask = pygame.mask.from_surface(self.sprite)
    
    def draw(self,win, offsetx,offsety):
        win.blit(self.sprite, (self.rect.x - offsetx, self.rect.y - offsety))    
```
### Clase Block 
Esta clase hereda de Object y representa un bloque en el juego. Tiene un método draw para dibujar el bloque en la ventana.

```    
class Block(Object):
    def __init__(self , x , y , size):
        super().__init__(x , y , size , size)
        block = get_block(size)
        self.image.blit(block,(0,0))
        self.mask = pygame.mask.from_surface(self.image)
```

### Genera el laberinto utilizando un autómata celular de Conway.

Comienza inicializando una matriz de celdas aleatorias y luego aplica reglas específicas para evolucionar el laberinto.
Al final, convierte las celdas vivas en caminos del laberinto.

``` 
def generate_maze(width, height):
```

### Inicializa la matriz del laberinto con todas las celdas como paredes

```
    maze = np.zeros((height, width))
 ```
    
### Inicializa la matriz de celulas vivas aleatorias

```
    // cells = np.random.randint(2, size=(height, width))
    cells = np.random.choice([0, 1], size=(N, N), p=[0.4, 0.6])
```
     
### Aplica el autómata celular de Conway para generar el laberinto

```
    for i in range(5):
        // Cuenta el número de vecinos vivos de cada celda
        neighbors = np.zeros((height, width))
        neighbors[1:, :] += cells[:-1, :]  # vecinos del norte
        neighbors[:-1, :] += cells[1:, :]  # vecinos del sur
        neighbors[:, 1:] += cells[:, :-1]  # vecinos del oeste
        neighbors[:, :-1] += cells[:, 1:]  # vecinos del este
        neighbors[1:, 1:] += cells[:-1, :-1]  # vecinos del noroeste
        neighbors[1:, :-1] += cells[:-1, 1:]  # vecinos del noreste
        neighbors[:-1, 1:] += cells[1:, :-1]  # vecinos del suroeste
        neighbors[:-1, :-1] += cells[1:, 1:]  # vecinos del sureste
```
    
### Aplica las reglas del autómata celular

 ```
        cells = np.logical_or(np.logical_and(cells == 1, neighbors < 2),
                              np.logical_and(cells == 1, neighbors > 3))
        cells = np.logical_or(cells, np.logical_and(cells == 0, neighbors == 3))
        
        // Convierte las celdas vivas en caminos del laberinto
        maze[cells == 1] = 1
```
    
### Coloca la entrada en uno de los extremos del laberinto
En esta sección específica del código, se implementa la lógica para colocar la entrada del laberinto en uno de los extremos. Dependiendo de cómo esté estructurado el código completo, esta parte puede variar, pero en general, el proceso implica lo siguiente:

* Se determina el tamaño del laberinto, es decir, el número de filas y columnas.
* Se elige aleatoriamente uno de los cuatro extremos del laberinto: arriba, abajo, izquierda o derecha.
En función del extremo seleccionado, se establece la posición de la entrada del laberinto. Por ejemplo, si se selecciona el extremo superior, la entrada se colocará en la primera fila en una columna aleatoria.
* Dependiendo de cómo esté representado el laberinto en el código, puede haber una estructura de datos que registre la posición de la entrada para su posterior uso.
    
```
    entry_side = random.choice(["top", "bottom", "left", "right"])
    if entry_side == "top":
        entry = (0, random.randint(0, width-1))
    elif entry_side == "bottom":
        entry = (height-1, random.randint(0, width-1))
    elif entry_side == "left":
        entry = (random.randint(0, height-1), 0)
    else:  # entry_side == "right"
        entry = (random.randint(0, height-1), width-1)
    maze[entry] = 2  # Marca la entrada
    // print(entry)
    // Coloca la salida en el extremo opuesto del laberinto
    if entry_side == "top":
        exit = (height-1, random.randint(0, width-1))
    elif entry_side == "bottom":
        exit = (0, random.randint(0, width-1))
    elif entry_side == "left":
        exit = (random.randint(0, height-1), width-1)
    else:  # entry_side == "right"
        exit = (random.randint(0, height-1), 0)
    maze[exit] = 3  # Marca la salida
    for i in range(height):
        for j in range(width):
            cells[i, j] = maze[i, j] > 0
    return entry, exit, cells, maze

def draw_maze(Ac_vacio,player,frutas,offsetx,offsety):
    object=[]
    a=len(Ac_vacio[0])
    for i in range (0,a) :
        for j in range (0,a) :
            if Ac_vacio[i][j]==0:
                if i==0:
                    if j==0:
                        object.append(Block(100,100,64))
                    else:
                        object.append(Block(100+64*j,100,64))
                else:
                    if j==0:
                        object.append(Block(100,100+64*i,64))
                    else:    
                        object.append(Block(100+64*j,100+64*i,64))
            elif Ac_vacio[i][j]==2:
                if i==0:
                    if j==0:
                        player.rect.x=100;player.rect.y=100
                    else:
                        player.rect.x=100+64*j;player.rect.y=100
                else:
                    if j==0:
                        player.rect.x=100;player.rect.y=100+64*i
                    else:
                        player.rect.x=100+64*j;player.rect.y=100+64*i
            elif Ac_vacio[i][j]==3:
                if i==0:
                    if j==0:
                        frutas.rect.x=100;frutas.rect.y=100
                    else:
                        frutas.rect.x=100+64*j ;frutas.rect.y=100
                else:
                    if j==0:
                        frutas.rect.x=100;frutas.rect.y=100+64*i
                    else:
                        frutas.rect.x=100+64*j;frutas.rect.y=100+64*i
    for i in range(0,a+1):
        object.append(Block(100-64,100-64,64)) #Primer Cubo
        object.append(Block(100-64,100+64*i,64)) #Pared Izquierda
        object.append(Block(100+64*i,100-64,64)) #Pared Superior
        object.append(Block(100+64*(a),100+64*i,64)) #Pared Derecha
        object.append(Block(100+64*i,100+64*(a),64)) #Paredd Inferior
    frutas.Frutas =random.choice(["Apple","Bananas","Cherries","Kiwi","Melon","Orange","Strawberry"])
    offsetx=player.rect.x-330;offsety=player.rect.y-330
    return offsetx,offsety,object

def draw(window,background,bg_image,offsetx,offsety,objects,player,fruta):
    for tile in background:
        window.blit(bg_image, tile)
    player.draw(window,offsetx,offsety)
    fruta.draw(window,offsetx,offsety)
    for obj in objects:
        obj.draw(window,offsetx,offsety)
    pygame.display.update()

def collide(player, objects, dx,dy):
        
    player.move(dx, dy)
    player.update()
    collided_object = None 
    for obj in objects:
        if pygame.sprite.collide_mask(player, obj):
            collided_object = obj 
            break
    player.move(-dx,-dy)
    player.update()
    return collided_object

def handle_move2(objects, player):
    if objects != None:
        player.x_vel = 0
        player.y_vel = 0
        keys = pygame.key.get_pressed()
        collide_left = collide(player, objects, -PLAYER_VEL * 2, 0)
        collide_right = collide(player, objects, PLAYER_VEL * 2, 0)
        collide_up = collide(player, objects, 0, -PLAYER_VEL * 2)
        collide_down = collide(player, objects, 0, PLAYER_VEL * 2)

        if (keys[pygame.K_a] or keys[pygame.K_LEFT]) and not collide_left :
         player.move_left(PLAYER_VEL)
        elif (keys[pygame.K_d] or keys[pygame.K_RIGHT]) and not collide_right:
         player.move_right(PLAYER_VEL)
        elif (keys[pygame.K_w] or keys[pygame.K_UP]) and not collide_up:
         player.move_up(PLAYER_VEL)
        elif (keys[pygame.K_s] or keys[pygame.K_DOWN]) and not collide_down:
         player.move_down(PLAYER_VEL)
    else:
        pass

def handle_move(objects,player, key):
    if objects!=None:
        player.x_vel = 0
        player.y_vel = 0
        if key == pygame.K_LEFT:
            player.move_left(PLAYER_VEL)
        elif key == pygame.K_RIGHT:
            player.move_right(PLAYER_VEL)
        elif key == pygame.K_UP:
            player.move_up(PLAYER_VEL)
        elif key == pygame.K_DOWN:
            player.move_down(PLAYER_VEL)
    else:
        pass
```

```
def check(x, y, N):
    return x < 0 or x >= N or y < 0 or y >= N
global mapa
mapa = {(0,-1) : pygame.K_LEFT,(0,1):pygame.K_RIGHT, (-1,0):pygame.K_UP, (1,0):pygame.K_DOWN}
dx = [1,-1,0,0]
dy = [0,0,1,-1]
def obtenerTeclasPresionar(ruta, x, y, N):
    move = []
    for i in range(4):
        if(check(x + dx[i], y + dy[i], N)): continue
        if ruta[x, y] == ruta[x + dx[i], y + dy[i]] - 1:
            # print(ruta[x,y], x,y,dx[i], dy[i],' - ',ruta[x + dx[i], y + dy[i]], x + dx[i], y + dy[i])
            move.append((dx[i], dy[i]))
            move += obtenerTeclasPresionar(ruta, x + dx[i], y + dy[i], N)
    return move
running = True
while running:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.MOUSEBUTTONDOWN:
            if not game_started:
                mouse_pos = pygame.mouse.get_pos()
                
                // Verificar si se hizo clic en una opción de dificultad
                for i in range(3):
                    if menu.rectDificultad[i].collidepoint(mouse_pos):
                        selected_difficulty = i
                
                // Verificar si se hizo clic en el botón de inicio
                if menu.rectBotonJugar.collidepoint(mouse_pos):
                    start_game()
    if not game_started:
        menu.draw_menu()
        pygame.display.flip()
    else:
```

### Lógica del juego
En esta parte del código, se implementa la lógica principal del juego, incluyendo las siguientes posibles funcionalidades:

Movimiento del jugador: Se establecen las reglas para que el jugador pueda moverse dentro del laberinto. Esto implica detectar las entradas del teclado o del ratón para recibir las acciones del jugador, y actualizar la posición del jugador en el laberinto en consecuencia.

* **Colisiones y límites:** Se implementa la lógica para detectar colisiones entre el jugador y las paredes del laberinto. Si el jugador intenta moverse hacia una pared, se le debe impedir avanzar en esa dirección. Además, se controla que el jugador no pueda salir del área del laberinto.

* **Objetivos y ganar el juego:** Se establecen las condiciones para que el jugador alcance el objetivo del juego. Esto podría ser llegar a una ubicación específica en el laberinto, recolectar objetos o resolver un rompecabezas. Cuando se cumple la condición de victoria, se muestra un mensaje de felicitaciones o se realiza alguna acción especial.

* **Interacción con elementos del laberinto:** Si el laberinto incluye elementos interactivos, como interruptores, puertas o ítems, se implementa la lógica para permitir al jugador interactuar con ellos. Esto puede incluir la activación de interruptores para abrir puertas, recoger objetos para obtener habilidades o solucionar rompecabezas específicos dentro del laberinto.

* **Sistema de puntuación:** Si el juego tiene un sistema de puntuación, se implementa la lógica para calcular y mostrar la puntuación del jugador. Esto puede basarse en factores como el tiempo que tarda en completar el laberinto, la cantidad de objetos recolectados o la eficiencia de los movimientos realizados.

```
        window = pygame.display.set_mode((WIDTH , HEIGHT))
        clock = pygame.time.Clock()
        background, bg_image = obtener_fondo("Brown.png")
        run = True
        player=Jugador(-100,-100,25,25)
        fruta=Fruit(-200,-200,25,30)
        offsetx = 0 
        offsety = 0
        scroll_area_width = 400
        scroll_area_height= 300
        objects=""
        N = (selected_difficulty + 1) * 10 
        inicio, final, laberinto, Acvacio = generate_maze(N,N)
        // print("ACvacio: ",Acvacio)
        laberinto = laberinto.astype(int)
        laberinto[inicio] = laberinto[final] = 1
        // inicio, final = np.asarray(inicio), np.asarray(final)
        IA = solved.IA_Maze(laberinto,N,inicio, final)
        ruta = IA.run()
        // move = IA.caminoPodado
        move = obtenerTeclasPresionar(ruta, inicio[0], inicio[1],N)
        // print(move)
        // print("Movimientos:\n",move)
        // print(ruta)
        offsetx,offsety,objects = draw_maze(Acvacio, player, fruta,offsetx, offsety)
        i = 0
        if len(move):
            print("IA ENCONTRO UNA SOLUCION")
            while len(move):
                // print(move[0], end="")
                for _ in range(16):
                    # time.sleep(1)
                    # print(_+1)
                    key = mapa[move[0]]
                    clock.tick(FPS)
                    fruta.loop(FPS)
                    player.loop(FPS)
                    handle_move(objects, player, key)
                    draw(window, background, bg_image, offsetx, offsety, objects, player, fruta)
                    if ((player.rect.right - offsetx >= WIDTH - scroll_area_width) and player.x_vel > 0) or (
                            (player.rect.left - offsetx <= scroll_area_width) and player.x_vel < 0):
                        offsetx += player.x_vel
                    elif ((player.rect.bottom - offsety >= HEIGHT - scroll_area_height) and player.y_vel > 0) or (
                            (player.rect.top - offsety <= scroll_area_height) and player.y_vel < 0):
                        offsety += player.y_vel
                move.pop(0)
            if pygame.sprite.collide_mask(player, fruta):
                fruta.Frutas = "Collected"
            for i in range(50):
                clock.tick(FPS)
                fruta.loop(FPS)
                // player.loop(FPS)
                draw(window, background, bg_image, offsetx, offsety, objects, player, fruta)
            // time.sleep(5)
        else:
            print("IA NO ENCONTRO UNA SOLUCION")
            while run:
                clock.tick(FPS)
                fruta.loop(FPS)
                player.loop(FPS)
                draw(window, background, bg_image, offsetx, offsety, objects, player, fruta)
                for event in pygame.event.get():
                    if event.type == pygame.QUIT:
                        run = False
                        break
                    if event.type == pygame.KEYDOWN:
                        if pygame.sprite.collide_mask(player, fruta):
                            fruta.Frutas = "Collected"
                            game_started = False
                handle_move2(objects, player)
                if ((player.rect.right - offsetx >= WIDTH - scroll_area_width) and player.x_vel > 0) or (
                        (player.rect.left - offsetx <= scroll_area_width) and player.x_vel < 0):
                    offsetx += player.x_vel
                elif ((player.rect.bottom - offsety >= HEIGHT - scroll_area_height) and player.y_vel > 0) or (
                        (player.rect.top - offsety <= scroll_area_height) and player.y_vel < 0):
                    offsety += player.y_vel
        game_started = False
        pygame.display.set_mode((WINDOW_WIDTH, WINDOW_HEIGHT))
   pygame.quit()
```

Este código muestra la estructura básica de un juego con Pygame, incluyendo la inicialización de la biblioteca, la creación de la ventana, la definición de objetos y personajes, y la lógica del juego. También incluye la generación de un laberinto utilizando un algoritmo específico.
