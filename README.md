
import pygame
import random
import sys
import os

# Cambiar el directorio de trabajo al del archivo actual
os.chdir(os.path.dirname(os.path.abspath(__file__)))

# Inicializar pygame
pygame.init()

# Dimensiones de la pantalla
ANCHO = 800
ALTO = 600

# Colores
BLANCO = (255, 255, 255)

# Configuración de la pantalla
pantalla = pygame.display.set_mode((ANCHO, ALTO))
pygame.display.set_caption("Space Shooter - Nave vs Asteroides")

# Reloj para controlar FPS
reloj = pygame.time.Clock()

# Cargar imágenes
imagen_nave = pygame.image.load("assets/nave3.jpg").convert_alpha()
imagen_asteroide = pygame.image.load("assets/asteroide4.png").convert_alpha()
imagen_fondo = pygame.image.load("assets/fondo3.jpg").convert()

# Escalar imágenes
imagen_nave = pygame.transform.scale(imagen_nave, (50, 40))
imagen_asteroide = pygame.transform.scale(imagen_asteroide, (50, 50))
imagen_fondo = pygame.transform.scale(imagen_fondo, (ANCHO, ALTO))

# Cargar sonidos
sonido_disparo = pygame.mixer.Sound("assets/disparo2.wav")
sonido_explosion = pygame.mixer.Sound("assets/explosion.wav")

# Clase para la nave
class Nave(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = imagen_nave
        self.rect = self.image.get_rect()
        self.rect.center = (ANCHO // 2, ALTO - 60)
        self.velocidad = 5

    def update(self):
        teclas = pygame.key.get_pressed()
        if teclas[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= self.velocidad
        if teclas[pygame.K_RIGHT] and self.rect.right < ANCHO:
            self.rect.x += self.velocidad

    def disparar(self):
        bala = Bala(self.rect.centerx, self.rect.top)
        todas_las_sprites.add(bala)
        balas.add(bala)
        sonido_disparo.play()

# Clase para las balas
class Bala(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.image = pygame.Surface((5, 10))
        self.image.fill(BLANCO)
        self.rect = self.image.get_rect()
        self.rect.center = (x, y)
        self.velocidad = -7

    def update(self):
        self.rect.y += self.velocidad
        if self.rect.bottom < 0:
            self.kill()

# Clase para los asteroides
class Asteroide(pygame.sprite.Sprite):
    def __init__(self):
        super().__init__()
        self.image = imagen_asteroide
        self.rect = self.image.get_rect()
        self.rect.x = random.randint(0, ANCHO - self.rect.width)
        self.rect.y = random.randint(-150, -40)
        self.velocidad = random.randint(2, 6)

    def update(self):
        self.rect.y += self.velocidad
        if self.rect.top > ALTO:
            self.rect.x = random.randint(0, ANCHO - self.rect.width)
            self.rect.y = random.randint(-150, -40)
            self.velocidad = random.randint(2, 6)

# Clase para las explosiones
class Explosion(pygame.sprite.Sprite):
    def __init__(self, x, y):
        super().__init__()
        self.frames = []
        for i in range(1, 6):  # Asume 5 imágenes para la explosión
            imagen = pygame.image.load(f"assets/explocion{i}.jpg").convert_alpha()
            imagen = pygame.transform.scale(imagen, (50, 50))
            self.frames.append(imagen)
        self.frame_actual = 0
        self.image = self.frames[self.frame_actual]
        self.rect = self.image.get_rect(center=(x, y))
        self.tiempo = pygame.time.get_ticks()

    def update(self):
        ahora = pygame.time.get_ticks()
        if ahora - self.tiempo > 50:  # Cambia de frame cada 50 ms
            self.tiempo = ahora
            self.frame_actual += 1
            if self.frame_actual < len(self.frames):
                self.image = self.frames[self.frame_actual]
            else:
                self.kill()  # Elimina la explosión al terminar la animación

# Función principal
def main():
    global todas_las_sprites, asteroides, balas
    # Grupos de sprites
    todas_las_sprites = pygame.sprite.Group()
    asteroides = pygame.sprite.Group()
    balas = pygame.sprite.Group()

    # Crear objetos
    nave = Nave()
    todas_las_sprites.add(nave)

    for _ in range(8):
        asteroide = Asteroide()
        todas_las_sprites.add(asteroide)
        asteroides.add(asteroide)

    # Puntuación
    puntuacion = 0
    fuente = pygame.font.SysFont("Arial", 24)

    # Bucle principal del juego
    jugando = True
    while jugando:
        for evento in pygame.event.get():
            if evento.type == pygame.QUIT:
                jugando = False
            elif evento.type == pygame.KEYDOWN:
                if evento.key == pygame.K_SPACE:
                    nave.disparar()

        # Actualizar
        todas_las_sprites.update()

        # Colisiones bala-asteroide
        colisiones = pygame.sprite.groupcollide(asteroides, balas, True, True)
        for colision in colisiones:
            puntuacion += 10
            sonido_explosion.play()
            nuevo_asteroide = Asteroide()
            todas_las_sprites.add(nuevo_asteroide)
            asteroides.add(nuevo_asteroide)

            # Crear una explosión
            explosion = Explosion(colision.rect.centerx, colision.rect.centery)
            todas_las_sprites.add(explosion)

        # Colisión asteroide-nave
        if pygame.sprite.spritecollideany(nave, asteroides):
            jugando = False

        # Dibujar
        pantalla.blit(imagen_fondo, (0, 0))
        todas_las_sprites.draw(pantalla)

        # Mostrar puntuación
        texto_puntuacion = fuente.render(f"Puntuación: {puntuacion}", True, BLANCO)
        pantalla.blit(texto_puntuacion, (10, 10))

        # Actualizar pantalla
        pygame.display.flip()

        # Controlar FPS
        reloj.tick(60)

        # Pantalla de Game Over
    pantalla.fill((0, 0, 0))  # Llenar la pantalla con color negro
    fuente_game_over = pygame.font.SysFont("Arial", 50)
    texto_game_over = fuente_game_over.render("GAME OVER", True, BLANCO)  # Texto en blanco
    pantalla.blit(texto_game_over, (ANCHO // 2 - texto_game_over.get_width() // 2, 
                                    ALTO // 2 - texto_game_over.get_height() // 2))  # Centrar texto
    pygame.display.flip()
    pygame.time.wait(3000)


    pygame.quit()
    sys.exit()

# Punto de entrada del programa
if __name__ == "__main__":
    main()
