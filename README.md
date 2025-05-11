import pygame
import random
import sys

pygame.init()


WIDTH, HEIGHT = 800, 600
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Військова битва")


BLACK = (0, 0, 0)
WHITE = (255, 255, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)


FPS = 60
clock = pygame.time.Clock()


PLAYER_IMG = pygame.image.load("player.png")
PLAYER_IMG = pygame.transform.scale(PLAYER_IMG, (50, 50))

ENEMY_IMG = pygame.image.load("enemy.png")
ENEMY_IMG = pygame.transform.scale(ENEMY_IMG, (50, 50))

BULLET_IMG = pygame.Surface((10, 5))
BULLET_IMG.fill(WHITE)


FONT = pygame.font.SysFont("comicsans", 40)

def draw_text(text, font, color, surface, x, y):
    text_obj = font.render(text, True, color)
    text_rect = text_obj.get_rect(center=(x, y))
    surface.blit(text_obj, text_rect)

def start_screen():
    while True:
        WIN.fill(BLACK)
        draw_text("Військова битва", FONT, WHITE, WIN, WIDTH // 2, HEIGHT // 2 - 50)
        draw_text("Натисни SPACE, щоб почати", FONT, WHITE, WIN, WIDTH // 2, HEIGHT // 2 + 50)
        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                return

def game_over_screen(score):
    while True:
        WIN.fill(BLACK)
        draw_text(f"Гра закінчена! Рахунок: {score}", FONT, WHITE, WIN, WIDTH // 2, HEIGHT // 2 - 50)
        draw_text("Натисни SPACE, щоб почати знову", FONT, WHITE, WIN, WIDTH // 2, HEIGHT // 2 + 50)
        pygame.display.update()

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN and event.key == pygame.K_SPACE:
                return

class Player:
    def __init__(self):
        self.image = PLAYER_IMG
        self.rect = self.image.get_rect(center=(WIDTH // 2, HEIGHT - 60))
        self.speed = 5
        self.lives = 3

    def draw(self):
        WIN.blit(self.image, self.rect)

    def move(self, keys):
        if keys[pygame.K_LEFT] and self.rect.left > 0:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT] and self.rect.right < WIDTH:
            self.rect.x += self.speed
        if keys[pygame.K_UP] and self.rect.top > 0:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN] and self.rect.bottom < HEIGHT:
            self.rect.y += self.speed

class Bullet:
    def __init__(self, x, y):
        self.rect = BULLET_IMG.get_rect(center=(x, y))
        self.speed = 10

    def move(self):
        self.rect.y -= self.speed

    def draw(self):
        WIN.blit(BULLET_IMG, self.rect)

class Enemy:
    def __init__(self):
        self.image = ENEMY_IMG
        self.rect = self.image.get_rect(center=(random.randint(50, WIDTH - 50), -50))
        self.speed = random.randint(2, 5)

    def move(self):
        self.rect.y += self.speed

    def draw(self):
        WIN.blit(self.image, self.rect)

def draw_window(player, bullets, enemies, score):
    WIN.fill(BLACK)
    player.draw()
    for bullet in bullets:
        bullet.draw()
    for enemy in enemies:
        enemy.draw()

    score_text = FONT.render(f"Рахунок: {score}", True, WHITE)
    WIN.blit(score_text, (10, 10))
    pygame.display.update()

def main():
    run = True
    player = Player()
    bullets = []
    enemies = []
    score = 0

    while run:
        clock.tick(FPS)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    bullet = Bullet(player.rect.centerx, player.rect.top)
                    bullets.append(bullet)

        keys = pygame.key.get_pressed()
        player.move(keys)

        
        if random.randint(1, 30) == 1:
            enemies.append(Enemy())

        
        for bullet in bullets[:]:
            bullet.move()
            if bullet.rect.bottom < 0:
                bullets.remove(bullet)

       
        for enemy in enemies[:]:
            enemy.move()
            if enemy.rect.top > HEIGHT:
                enemies.remove(enemy)

   
        for enemy in enemies[:]:
            if player.rect.colliderect(enemy.rect):
                player.lives -= 1
                enemies.remove(enemy)
                if player.lives == 0:
                    game_over_screen(score)
                    main()
            for bullet in bullets[:]:
                if bullet.rect.colliderect(enemy.rect):
                    bullets.remove(bullet)
                    enemies.remove(enemy)
                    score += 1

        draw_window(player, bullets, enemies, score)

if name == "__main__":
    start_screen()
    main()
