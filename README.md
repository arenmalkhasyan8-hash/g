import pygame
import random
import sys

pygame.init()

WIDTH, HEIGHT = 1100, 750
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("GAME CENTER CLICK")

FONT = pygame.font.SysFont("Arial", 40)

WHITE = (255,255,255)
BLACK = (0,0,0)
GREEN = (0,200,0)
BLUE = (0,0,200)
RED = (200,0,0)

# -------- BUTTON CLASS --------
class Button:
    def __init__(self, text, x, y, w, h):
        self.text = text
        self.rect = pygame.Rect(x, y, w, h)

    def draw(self):
        pygame.draw.rect(WIN, GREEN, self.rect)
        label = FONT.render(self.text, True, BLACK)
        WIN.blit(label, (self.rect.x+20, self.rect.y+10))

    def is_clicked(self, pos):
        return self.rect.collidepoint(pos)

# -------- MENU --------
def menu():
    buttons = [
        Button("DODGE", 400, 150, 300, 60),
        Button("MEMORY", 400, 230, 300, 60),
        Button("BRICKS", 400, 310, 300, 60),
        Button("MAZE", 400, 390, 300, 60),
        Button("SNAKE", 400, 470, 300, 60),
    ]

    while True:
        WIN.fill(BLACK)

        for b in buttons:
            b.draw()

        pygame.display.update()

        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if e.type == pygame.MOUSEBUTTONDOWN:
                pos = pygame.mouse.get_pos()

                if buttons[0].is_clicked(pos): dodge()
                if buttons[1].is_clicked(pos): memory()
                if buttons[2].is_clicked(pos): bricks()
                if buttons[3].is_clicked(pos): maze()
                if buttons[4].is_clicked(pos): snake()

# -------- DODGE --------
def dodge():
    player = pygame.Rect(500, 650, 50, 50)
    enemies = []
    clock = pygame.time.Clock()

    while True:
        clock.tick(60)
        WIN.fill(BLACK)

        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit(); sys.exit()

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]: player.x -= 7
        if keys[pygame.K_RIGHT]: player.x += 7

        if random.randint(1,25) == 1:
            enemies.append(pygame.Rect(random.randint(0, WIDTH-50), 0, 50, 50))

        for en in enemies:
            en.y += 6
            pygame.draw.rect(WIN, RED, en)
            if player.colliderect(en):
                return

        pygame.draw.rect(WIN, BLUE, player)
        pygame.display.update()

# -------- MEMORY --------
def memory():
    size = 4
    values = list(range(1,9))*2
    random.shuffle(values)

    grid = [[values.pop() for _ in range(size)] for _ in range(size)]
    revealed = [[False]*size for _ in range(size)]
    first = None

    clock = pygame.time.Clock()

    while True:
        clock.tick(60)
        WIN.fill(WHITE)

        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit(); sys.exit()

            if e.type == pygame.MOUSEBUTTONDOWN:
                x,y = pygame.mouse.get_pos()
                i,j = y//150, x//150

                if i < size and j < size:
                    if not revealed[i][j]:
                        revealed[i][j] = True
                        if first is None:
                            first = (i,j)
                        else:
                            if grid[i][j] != grid[first[0]][first[1]]:
                                pygame.time.delay(500)
                                revealed[i][j] = False
                                revealed[first[0]][first[1]] = False
                            first = None

        for i in range(size):
            for j in range(size):
                rect = pygame.Rect(j*150, i*150, 140, 140)
                pygame.draw.rect(WIN, BLUE if revealed[i][j] else BLACK, rect)

                if revealed[i][j]:
                    text = FONT.render(str(grid[i][j]), True, WHITE)
                    WIN.blit(text, (j*150+50, i*150+50))

        pygame.display.update()

# -------- BRICKS --------
def bricks():
    paddle = pygame.Rect(500, 700, 120, 20)
    ball = pygame.Rect(550, 400, 20, 20)
    dx, dy = 5, -5

    bricks = [pygame.Rect(j*100+10, i*40+10, 80, 30) for i in range(5) for j in range(10)]
    clock = pygame.time.Clock()

    while True:
        clock.tick(60)
        WIN.fill(BLACK)

        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit(); sys.exit()

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]: paddle.x -= 8
        if keys[pygame.K_RIGHT]: paddle.x += 8

        ball.x += dx
        ball.y += dy

        if ball.left <= 0 or ball.right >= WIDTH: dx *= -1
        if ball.top <= 0: dy *= -1
        if ball.colliderect(paddle): dy *= -1

        for b in bricks[:]:
            if ball.colliderect(b):
                bricks.remove(b)
                dy *= -1

        if ball.bottom > HEIGHT:
            return

        pygame.draw.rect(WIN, GREEN, paddle)
        pygame.draw.ellipse(WIN, WHITE, ball)

        for b in bricks:
            pygame.draw.rect(WIN, RED, b)

        pygame.display.update()

# -------- MAZE --------
def maze():
    player = pygame.Rect(50, 50, 30, 30)
    goal = pygame.Rect(1000, 650, 50, 50)

    walls = [pygame.Rect(random.randint(0, WIDTH), random.randint(0, HEIGHT), 120, 20) for _ in range(25)]
    clock = pygame.time.Clock()

    while True:
        clock.tick(60)
        WIN.fill(WHITE)

        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit(); sys.exit()

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]: player.x -= 5
        if keys[pygame.K_RIGHT]: player.x += 5
        if keys[pygame.K_UP]: player.y -= 5
        if keys[pygame.K_DOWN]: player.y += 5

        for w in walls:
            pygame.draw.rect(WIN, BLACK, w)
            if player.colliderect(w):
                player.topleft = (50,50)

        pygame.draw.rect(WIN, GREEN, goal)

        if player.colliderect(goal):
            return

        pygame.draw.rect(WIN, BLUE, player)
        pygame.display.update()

# -------- SNAKE --------
def snake():
    snake = [(200,200)]
    dx, dy = 20, 0
    food = (random.randint(0, 50)*20, random.randint(0, 35)*20)

    clock = pygame.time.Clock()

    while True:
        clock.tick(10)
        WIN.fill(BLACK)

        for e in pygame.event.get():
            if e.type == pygame.QUIT:
                pygame.quit(); sys.exit()

        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]: dx,dy = -20,0
        if keys[pygame.K_RIGHT]: dx,dy = 20,0
        if keys[pygame.K_UP]: dx,dy = 0,-20
        if keys[pygame.K_DOWN]: dx,dy = 0,20

        head = (snake[0][0]+dx, snake[0][1]+dy)
        snake.insert(0, head)

        if head == food:
            food = (random.randint(0, 50)*20, random.randint(0, 35)*20)
        else:
            snake.pop()

        if head[0]<0 or head[0]>WIDTH or head[1]<0 or head[1]>HEIGHT:
            return

        for s in snake:
            pygame.draw.rect(WIN, GREEN, (*s,20,20))

        pygame.draw.rect(WIN, RED, (*food,20,20))
        pygame.display.update()

# -------- RUN --------
menu()
