import pygame
import sys
import random

# إعدادات اللعبة
WIDTH, HEIGHT = 800, 600
FPS = 60

# ألوان
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
RED = (255, 0, 0)
GREEN = (0, 255, 0)

# إعداد Pygame
pygame.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("لعبة حياة و قتال")
clock = pygame.time.Clock()
font = pygame.font.SysFont("arial", 24)

# اللاعب
player = pygame.Rect(100, 100, 50, 50)
player_speed = 5
player_health = 100

# الرصاص
bullets = []
bullet_speed = 10

# أعداء
enemies = []
enemy_spawn_timer = 0

# موارد (Health Packs)
health_packs = []
health_timer = 0

def draw_text(text, x, y, color=BLACK):
    label = font.render(text, True, color)
    screen.blit(label, (x, y))

def draw_player():
    pygame.draw.rect(screen, BLUE, player)

def draw_bullets():
    for bullet in bullets:
        pygame.draw.rect(screen, RED, bullet)

def draw_enemies():
    for enemy in enemies:
        pygame.draw.rect(screen, (150, 0, 0), enemy)

def draw_health_packs():
    for pack in health_packs:
        pygame.draw.rect(screen, GREEN, pack)

# حلقة اللعبة
running = True
while running:
    screen.fill(WHITE)
    clock.tick(FPS)

    # الأحداث
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

    # تحكم اللاعب
    keys = pygame.key.get_pressed()
    if keys[pygame.K_LEFT]: player.x -= player_speed
    if keys[pygame.K_RIGHT]: player.x += player_speed
    if keys[pygame.K_UP]: player.y -= player_speed
    if keys[pygame.K_DOWN]: player.y += player_speed
    if keys[pygame.K_SPACE]:
        if len(bullets) < 5:  # تحديد عدد الطلقات في نفس الوقت
            bullets.append(pygame.Rect(player.centerx, player.top, 5, 10))

    # تحريك الرصاص
    for bullet in bullets[:]:
        bullet.y -= bullet_speed
        if bullet.y < 0:
            bullets.remove(bullet)

    # توليد الأعداء
    enemy_spawn_timer += 1
    if enemy_spawn_timer > 60:
        enemy = pygame.Rect(random.randint(0, WIDTH-40), -40, 40, 40)
        enemies.append(enemy)
        enemy_spawn_timer = 0

    # تحريك الأعداء
    for enemy in enemies[:]:
        enemy.y += 2
        if enemy.colliderect(player):
            player_health -= 10
            enemies.remove(enemy)
        for bullet in bullets:
            if enemy.colliderect(bullet):
                try:
                    enemies.remove(enemy)
                    bullets.remove(bullet)
                except:
                    pass

    # توليد الموارد
    health_timer += 1
    if health_timer > 300:
        pack = pygame.Rect(random.randint(0, WIDTH-30), random.randint(0, HEIGHT-30), 30, 30)
        health_packs.append(pack)
        health_timer = 0

    # جمع الموارد
    for pack in health_packs[:]:
        if player.colliderect(pack):
            player_health = min(100, player_health + 20)
            health_packs.remove(pack)

    # رسم العناصر
    draw_player()
    draw_bullets()
    draw_enemies()
    draw_health_packs()

    draw_text(f"الطاقة: {player_health}", 10, 10)

    if player_health <= 0:
        draw_text("💀 خسرت! اضغط Esc للخروج", WIDTH // 3, HEIGHT // 2, RED)
        pygame.display.update()
        keys = pygame.key.get_pressed()
        if keys[pygame.K_ESCAPE]:
            break
        continue

    pygame.display.flip()

pygame.quit()
sys.exit()
