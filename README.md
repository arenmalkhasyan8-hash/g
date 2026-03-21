import pygame
import sys
import random
import copy

pygame.init()

WIDTH = 540
WIN = pygame.display.set_mode((WIDTH, WIDTH))
pygame.display.set_caption("Sudoku Random")

WHITE = (255,255,255)
BLACK = (0,0,0)
BLUE = (0,0,255)
RED = (255,120,120)

font = pygame.font.SysFont(None, 45)
menu_font = pygame.font.SysFont(None, 60)

# ---- Լուծում (մշտական ամբողջական sudoku) ----
solution = [
    [5,3,4,6,7,8,9,1,2],
    [6,7,2,1,9,5,3,4,8],
    [1,9,8,3,4,2,5,6,7],
    [8,5,9,7,6,1,4,2,3],
    [4,2,6,8,5,3,7,9,1],
    [7,1,3,9,2,4,8,5,6],
    [9,6,1,5,3,7,2,8,4],
    [2,8,7,4,1,9,6,3,5],
    [3,4,5,2,8,6,1,7,9]
]

board = []
start_board = []
selected = None
game_state = "menu"

# ---- Ռանդոմ պազլ ստեղծել ----
def generate_puzzle(level):
    puzzle = copy.deepcopy(solution)

    if level == 1:
        remove_count = 35   # հեշտ
    else:
        remove_count = 50   # դժվար

    removed = 0
    while removed < remove_count:
        row = random.randint(0,8)
        col = random.randint(0,8)
        if puzzle[row][col] != 0:
            puzzle[row][col] = 0
            removed += 1

    return puzzle

# ---- Ստուգում ----
def is_valid(num, row, col):
    for i in range(9):
        if board[row][i] == num and i != col:
            return False

    for i in range(9):
        if board[i][col] == num and i != row:
            return False

    box_x = col // 3
    box_y = row // 3

    for i in range(box_y*3, box_y*3+3):
        for j in range(box_x*3, box_x*3+3):
            if board[i][j] == num and (i,j) != (row,col):
                return False

    return True

# ---- Նկարել ----
def draw_menu():
    WIN.fill(WHITE)
    t1 = menu_font.render("Choose Level", True, BLACK)
    t2 = font.render("Press 1 - Level 1", True, BLACK)
    t3 = font.render("Press 2 - Level 2", True, BLACK)

    WIN.blit(t1, (WIDTH//2 - 150, 150))
    WIN.blit(t2, (WIDTH//2 - 130, 260))
    WIN.blit(t3, (WIDTH//2 - 130, 320))
    pygame.display.update()

def draw_game():
    WIN.fill(WHITE)
    gap = WIDTH // 9

    for row in range(9):
        for col in range(9):

            value = board[row][col]

            if value != 0:

                if not is_valid(value, row, col):
                    pygame.draw.rect(WIN, RED, (col*gap, row*gap, gap, gap))

                text = font.render(str(value), True, BLACK)
                WIN.blit(text, (col*gap + 20, row*gap + 15))

    for i in range(10):
        thick = 4 if i % 3 == 0 else 1
        pygame.draw.line(WIN, BLACK, (0, i*gap), (WIDTH, i*gap), thick)
        pygame.draw.line(WIN, BLACK, (i*gap, 0), (i*gap, WIDTH), thick)

    if selected:
        row, col = selected
        pygame.draw.rect(WIN, BLUE, (col*gap, row*gap, gap, gap), 3)

    pygame.display.update()

# ---- Game Loop ----
running = True
while running:

    if game_state == "menu":
        draw_menu()
    else:
        draw_game()

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if game_state == "menu":
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    start_board = generate_puzzle(1)
                if event.key == pygame.K_2:
                    start_board = generate_puzzle(2)

                if event.key in [pygame.K_1, pygame.K_2]:
                    board = copy.deepcopy(start_board)
                    game_state = "game"

        elif game_state == "game":

            if event.type == pygame.MOUSEBUTTONDOWN:
                x, y = pygame.mouse.get_pos()
                gap = WIDTH // 9
                selected = (y // gap, x // gap)

            if event.type == pygame.KEYDOWN:
                if selected:
                    row, col = selected

                    if start_board[row][col] == 0:
                        if event.unicode.isdigit():
                            num = int(event.unicode)
                            if 1 <= num <= 9:
                                board[row][col] = num

                        if event.key == pygame.K_BACKSPACE:
                            board[row][col] = 0S
