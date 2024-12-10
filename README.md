# SpaceJetpack
To RUN the game, follow the following steps :-
1). open the windows powershell terminal and execute the following code to access the file path where the ZIP Files are downloaded
    cd "file path of the ZIP Files"
2). type the following code in the windows powershell terminal after following the first step 
    python space_jetpack.py
the game will begin.

/* if the game throws an error, or fails to run, make sure the correct paths have been used in the source code. */

import pygame
import random
import csv

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Space Jetpack Adventure")

# Colors
BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)

# FPS
FPS = 60
clock = pygame.time.Clock()

# Background music
pygame.mixer.init()
pygame.mixer.music.load(r"D:/projects/study/REVA/jetpack/interstellar_theme.mp3")  # Replace with the actual path to the file
pygame.mixer.music.play(-1)  # Loop the music indefinitely

# Load assets
astronaut_image = pygame.image.load(r"D:\projects\study\REVA\jetpack\jetpackstronaut.png")  # Replace with the actual path to the astronaut image
astronaut_image = pygame.transform.scale(astronaut_image, (50, 50))
space_debris_image = pygame.image.load(r"D:/projects/study/REVA/jetpack/space_debris.png")  # Replace with the actual path to the debris image
space_debris_image = pygame.transform.scale(space_debris_image, (50, 200))
galaxy_background = pygame.image.load(r"D:/projects/study/REVA/jetpack/galaxy_background.jpg")  # Replace with the actual path to the background image
galaxy_background = pygame.transform.scale(galaxy_background, (WIDTH, HEIGHT))

# Player
player_size = 50
player_x = 100
player_y = HEIGHT // 2
player_y_velocity = 0
player_gravity = 0.5
player_jump_strength = -10

# Obstacles
obstacle_width = 50
obstacle_height = 200
obstacle_speed = 5
obstacles = []
obstacle_spawn_rate = 1500  # in milliseconds
pygame.time.set_timer(pygame.USEREVENT, obstacle_spawn_rate)

# Game variables
score = 0
font = pygame.font.Font(None, 36)
speed_increase_interval = 10  # Score interval to increase speed
leaderboard_file = r"D:/projects/study/REVA/jetpack/leaderboard.csv"

# Leaderboard setup
try:
    with open(leaderboard_file, "x", newline="") as file:
        writer = csv.writer(file)
        writer.writerow(["Name", "Score"])
except FileExistsError:
    pass

def draw_player(x, y):
    screen.blit(astronaut_image, (x, y))

def draw_obstacle(x, y):
    screen.blit(space_debris_image, (x, y))

def display_score(score):
    score_text = font.render(f"Score: {score}", True, WHITE)
    screen.blit(score_text, (10, 10))

def save_to_leaderboard(name, score):
    with open(leaderboard_file, "a", newline="") as file:
        writer = csv.writer(file)
        writer.writerow([name, score])

def get_highest_score():
    highest_score = 0
    highest_scorer = ""
    with open(leaderboard_file, "r") as file:
        reader = csv.reader(file)
        next(reader)  # Skip header
        for row in reader:
            name, score = row
            if int(score) > highest_score:
                highest_score = int(score)
                highest_scorer = name
    return highest_scorer, highest_score

def game_over_screen(score, highest_scorer, highest_score):
    screen.blit(galaxy_background, (0, 0))
    game_over_text = font.render("GAME OVER", True, RED)
    score_text = font.render(f"Your Score: {score}", True, WHITE)
    highest_score_text = font.render(f"Highest Score: {highest_score} by {highest_scorer}", True, WHITE)
    screen.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 2 - 50))
    screen.blit(score_text, (WIDTH // 2 - score_text.get_width() // 2, HEIGHT // 2))
    screen.blit(highest_score_text, (WIDTH // 2 - highest_score_text.get_width() // 2, HEIGHT // 2 + 50))
    pygame.display.flip()
    pygame.time.wait(3000)

def starting_screen():
    while True:
        screen.blit(galaxy_background, (0, 0))
        title_text = font.render("Space Jetpack Adventure", True, WHITE)
        single_player_text = font.render("Single Player", True, WHITE)
        multiplayer_text = font.render("Multiplayer", True, WHITE)

        screen.blit(title_text, (WIDTH // 2 - title_text.get_width() // 2, HEIGHT // 2 - 100))
        single_player_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 - 20, 200, 50)
        multiplayer_button = pygame.Rect(WIDTH // 2 - 100, HEIGHT // 2 + 50, 200, 50)

        pygame.draw.rect(screen, RED, single_player_button)
        pygame.draw.rect(screen, RED, multiplayer_button)

        screen.blit(single_player_text, (WIDTH // 2 - single_player_text.get_width() // 2, HEIGHT // 2 - 10))
        screen.blit(multiplayer_text, (WIDTH // 2 - multiplayer_text.get_width() // 2, HEIGHT // 2 + 60))

        pygame.display.flip()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                quit()
            if event.type == pygame.MOUSEBUTTONDOWN:
                if single_player_button.collidepoint(event.pos):
                    return "single"
                if multiplayer_button.collidepoint(event.pos):
                    return "multi"

def play_game(player_name):
    global obstacles, score, obstacle_speed
    player_y = HEIGHT // 2
    player_y_velocity = 0
    obstacles = []
    score = 0
    obstacle_speed = 5
    running = True

    while running:
        screen.blit(galaxy_background, (0, 0))

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    player_y_velocity = player_jump_strength
            if event.type == pygame.USEREVENT:
                obstacle_x = WIDTH
                obstacle_y = random.randint(0, HEIGHT - obstacle_height)
                obstacles.append([obstacle_x, obstacle_y])  # Store as a list for mutability

        # Player movement
        player_y_velocity += player_gravity
        player_y += player_y_velocity

        # Prevent the player from going off-screen
        if player_y < 0:
            player_y = 0
            player_y_velocity = 0
        if player_y > HEIGHT - player_size:
            player_y = HEIGHT - player_size
            player_y_velocity = 0

        draw_player(player_x, player_y)

        # Obstacle movement and collision detection
        for obstacle in obstacles[:]:
            obstacle[0] -= obstacle_speed

            if obstacle[0] + obstacle_width < 0:
                obstacles.remove(obstacle)
                score += 1

                # Increase speed after reaching a multiple of the interval
                if score % speed_increase_interval == 0:
                    obstacle_speed += 1

            # Check for collisions
            if (player_x < obstacle[0] + obstacle_width and
                player_x + player_size > obstacle[0] and
                player_y < obstacle[1] + obstacle_height and
                player_y + player_size > obstacle[1]):
                running = False

            draw_obstacle(obstacle[0], obstacle[1])

        display_score(score)

        pygame.display.flip()
        clock.tick(FPS)

    return score

# Main program
mode = starting_screen()

if mode == "single":
    player_name = input("Enter your name: ")
    score = play_game(player_name)
    save_to_leaderboard(player_name, score)
    highest_scorer, highest_score = get_highest_score()
    game_over_screen(score, highest_scorer, highest_score)

elif mode == "multi":
    player1_name = input("Enter Player 1's name: ")
    player2_name = input("Enter Player 2's name: ")

    print(f"{player1_name}'s turn!")
    player1_score = play_game(player1_name)

    print(f"{player2_name}'s turn!")
    player2_score = play_game(player2_name)

    save_to_leaderboard(player1_name, player1_score)
    save_to_leaderboard(player2_name, player2_score)

    highest_scorer, highest_score = get_highest_score()
    screen.blit(galaxy_background, (0, 0))
    player1_score_text = font.render(f"{player1_name}'s Score: {player1_score}", True, WHITE)
    player2_score_text = font.render(f"{player2_name}'s Score: {player2_score}", True, WHITE)
    highest_score_text = font.render(f"Highest Score: {highest_score} by {highest_scorer}", True, WHITE)

    screen.blit(player1_score_text, (WIDTH // 2 - player1_score_text.get_width() // 2, HEIGHT // 2 - 50))
    screen.blit(player2_score_text, (WIDTH // 2 - player2_score_text.get_width() // 2, HEIGHT // 2))
    screen.blit(highest_score_text, (WIDTH // 2 - highest_score_text.get_width() // 2, HEIGHT // 2 + 50))

    pygame.display.flip()
    pygame.time.wait(5000)

pygame.quit()
