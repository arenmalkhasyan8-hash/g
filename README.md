import random

SIZE = 4

def new_grid():
    return [[0]*SIZE for _ in range(SIZE)]

def add_number(grid):
    empty = [(i, j) for i in range(SIZE) for j in range(SIZE) if grid[i][j] == 0]
    if empty:
        i, j = random.choice(empty)
        grid[i][j] = 2 if random.random() < 0.9 else 4

def print_grid(grid):
    print("\n" + "-"*25)
    for row in grid:
        for num in row:
            if num == 0:
                print(" . ", end="")
            else:
                print(f"{num:4}", end="")
        print()
    print("-"*25)

def compress(row):
    new = [i for i in row if i != 0]
    new += [0] * (SIZE - len(new))
    return new

def merge(row):
    for i in range(SIZE-1):
        if row[i] == row[i+1] and row[i] != 0:
            row[i] *= 2
            row[i+1] = 0
    return row

def move_left(grid):
    changed = False
    new_grid = []
    for row in grid:
        c = compress(row)
        m = merge(c)
        c2 = compress(m)
        if c2 != row:
            changed = True
        new_grid.append(c2)
    return new_grid, changed

def reverse(grid):
    return [row[::-1] for row in grid]

def transpose(grid):
    return [list(row) for row in zip(*grid)]

def move_right(grid):
    rev = reverse(grid)
    new, changed = move_left(rev)
    return reverse(new), changed

def move_up(grid):
    t = transpose(grid)
    new, changed = move_left(t)
    return transpose(new), changed

def move_down(grid):
    t = transpose(grid)
    new, changed = move_right(t)
    return transpose(new), changed

def game_over(grid):
    for i in range(SIZE):
        for j in range(SIZE):
            if grid[i][j] == 0:
                return False
            if i < SIZE-1 and grid[i][j] == grid[i+1][j]:
                return False
            if j < SIZE-1 and grid[i][j] == grid[i][j+1]:
                return False
    return True

grid = new_grid()
add_number(grid)
add_number(grid)

while True:
    print_grid(grid)
    move = input("Move (W=up, A=right, S=left, X=down, Q=quit): ").lower()

    if move == "q":
        break
    elif move == "s":     # ձախ
        grid, changed = move_left(grid)
    elif move == "a":     # աջ
        grid, changed = move_right(grid)
    elif move == "w":     # վերև
        grid, changed = move_up(grid)
    elif move == "x":     # ներքև
        grid, changed = move_down(grid)
    else:
        continue

    if changed:
        add_number(grid)

    if game_over(grid):
        print_grid(grid)
        print("GAME OVER!")
        break

