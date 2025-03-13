import pygame
import random

# Configuración del juego
WIDTH, HEIGHT = 600, 400
GRID_SIZE = 20
BLACK, WHITE, RED = (0, 0, 0), (255, 255, 255), (255, 0, 0)
COLORS = [(0, 255, 0), (0, 0, 255), (255, 255, 0), (255, 165, 0), (128, 0, 128)]

# Inicializar pygame
pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 36)

# Función para generar comida
def generate_food():
    return [random.randint(0, (WIDTH // GRID_SIZE) - 1) * GRID_SIZE,
            random.randint(0, (HEIGHT // GRID_SIZE) - 1) * GRID_SIZE]

# Dibujar estrellas de fondo
def draw_stars():
    for _ in range(50):
        x, y = random.randint(0, WIDTH), random.randint(0, HEIGHT)
        pygame.draw.circle(screen, WHITE, (x, y), 1)

# Clase de la serpiente
class Snake:
    def __init__(self):
        self.body = [[100, 100]]
        self.direction = [GRID_SIZE, 0]
        self.growing = False
        self.color = random.choice(COLORS)
    
    def move(self):
        head = [self.body[0][0] + self.direction[0], self.body[0][1] + self.direction[1]]
        
        if head in self.body or head[0] < 0 or head[0] >= WIDTH or head[1] < 0 or head[1] >= HEIGHT:
            return False
        
        self.body.insert(0, head)
        if not self.growing:
            self.body.pop()
        else:
            self.growing = False
        return True
    
    def grow(self):
        self.growing = True
        self.color = random.choice(COLORS)
    
    def change_direction(self, new_direction):
        if (new_direction[0] != -self.direction[0] and new_direction[1] != -self.direction[1]):
            self.direction = new_direction
    
    def draw(self):
        for segment in self.body:
            pygame.draw.rect(screen, self.color, (*segment, GRID_SIZE, GRID_SIZE))

# Función de reinicio
def reset_game():
    global snake, food, score, running
    snake = Snake()
    food = generate_food()
    score = 0
    running = True

# Inicializar juego
snake = Snake()
food = generate_food()
score = 0
running = True

# Bucle del juego
while running:
    screen.fill(BLACK)
    draw_stars()
    
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False
        elif event.type == pygame.KEYDOWN:
            if event.key == pygame.K_UP:
                snake.change_direction([0, -GRID_SIZE])
            elif event.key == pygame.K_DOWN:
                snake.change_direction([0, GRID_SIZE])
            elif event.key == pygame.K_LEFT:
                snake.change_direction([-GRID_SIZE, 0])
            elif event.key == pygame.K_RIGHT:
                snake.change_direction([GRID_SIZE, 0])
    
    if not snake.move():
        screen.fill(BLACK)
        text = font.render(f"Game Over! Score: {score}. Press R to Restart", True, WHITE)
        screen.blit(text, (WIDTH // 6, HEIGHT // 2))
        pygame.display.flip()
        waiting = True
        while waiting:
            for event in pygame.event.get():
                if event.type == pygame.QUIT:
                    running = False
                    waiting = False
                elif event.type == pygame.KEYDOWN and event.key == pygame.K_r:
                    reset_game()
                    waiting = False
        continue
    
    if snake.body[0] == food:
        snake.grow()
        food = generate_food()
        score += 10
    
    pygame.draw.rect(screen, RED, (*food, GRID_SIZE, GRID_SIZE))
    snake.draw()
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))
    pygame.display.flip()
    clock.tick(10)

pygame.quit()
