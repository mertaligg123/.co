python
Kodu kopyala
import pygame
import random

# Pygame'i başlat
pygame.init()

# Ekran boyutları
screen_width = 400
screen_height = 600
screen = pygame.display.set_mode((screen_width, screen_height))
pygame.display.set_caption("Flappy Bird")

# Renkler
WHITE = (255, 255, 255)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)

# Kuş ayarları
bird_width = 50
bird_height = 50
bird_x = 100
bird_y = screen_height // 2
bird_velocity = 0
gravity = 0.5
jump_strength = -10

# Boru ayarları
pipe_width = 60
pipe_gap = 150
pipe_velocity = 3
pipes = []

# Skor
score = 0
font = pygame.font.SysFont('Arial', 30)

# Saat ayarı
clock = pygame.time.Clock()

# Kuşı hareket ettiren fonksiyon
def draw_bird(y):
    pygame.draw.rect(screen, BLUE, (bird_x, y, bird_width, bird_height))

# Boruları çizme fonksiyonu
def draw_pipes():
    global pipes
    for pipe in pipes:
        pygame.draw.rect(screen, GREEN, pipe['top_rect'])
        pygame.draw.rect(screen, GREEN, pipe['bottom_rect'])

# Yeni boru ekleme fonksiyonu
def add_pipe():
    pipe_height = random.randint(100, screen_height - pipe_gap - 100)
    top_rect = pygame.Rect(screen_width, 0, pipe_width, pipe_height)
    bottom_rect = pygame.Rect(screen_width, pipe_height + pipe_gap, pipe_width, screen_height - pipe_height - pipe_gap)
    pipes.append({'top_rect': top_rect, 'bottom_rect': bottom_rect})

# Boruları hareket ettirme fonksiyonu
def move_pipes():
    global pipes, score
    for pipe in pipes:
        pipe['top_rect'].x -= pipe_velocity
        pipe['bottom_rect'].x -= pipe_velocity

    # Boruları kaldırma
    pipes = [pipe for pipe in pipes if pipe['top_rect'].x + pipe_width > 0]

    # Skoru arttırma
    for pipe in pipes:
        if pipe['top_rect'].x + pipe_width < bird_x and not pipe.get('scored', False):
            score += 1
            pipe['scored'] = True

    return pipes

# Oyun sona erme kontrolü
def check_collision():
    global bird_y, pipes
    # Kuşun ekran dışına çıkması
    if bird_y <= 0 or bird_y + bird_height >= screen_height:
        return True

    # Borularla çarpışma
    bird_rect = pygame.Rect(bird_x, bird_y, bird_width, bird_height)
    for pipe in pipes:
        if bird_rect.colliderect(pipe['top_rect']) or bird_rect.colliderect(pipe['bottom_rect']):
            return True

    return False

# Buton sınıfı
class Button:
    def __init__(self, x, y, width, height, color, text):
        self.rect = pygame.Rect(x, y, width, height)
        self.color = color
        self.text = text
        self.font = pygame.font.SysFont('Arial', 30)
        
    def draw(self, screen):
        pygame.draw.rect(screen, self.color, self.rect)
        text_surface = self.font.render(self.text, True, BLACK)
        screen.blit(text_surface, (self.rect.x + (self.rect.width - text_surface.get_width()) // 2, 
                                  self.rect.y + (self.rect.height - text_surface.get_height()) // 2))
    
    def is_clicked(self, pos):
        if self.rect.collidepoint(pos):
            return True
        return False

# Yeniden başlatma fonksiyonu
def restart_game():
    global bird_y, bird_velocity, pipes, score
    bird_y = screen_height // 2
    bird_velocity = 0
    pipes = []
    score = 0
    add_pipe()

# Oyun bittiğinde çıkan mesaj ve buton
def game_over_message():
    game_over_font = pygame.font.SysFont('Arial', 50)
    message = game_over_font.render("Game Over!", True, RED)
    screen.blit(message, (screen_width // 4, screen_height // 3))

    restart_button = Button(screen_width // 3, screen_height // 2, 150, 50, (0, 255, 0), "Restart")
    restart_button.draw(screen)
    
    pygame.display.update()

    return restart_button

# Ana oyun fonksiyonu
def game_loop():
    global bird_y, bird_velocity, pipes, score

    running = True
    bird_y = screen_height // 2
    bird_velocity = 0
    pipes = []
    score = 0
    add_pipe()

    while running:
        screen.fill(WHITE)

        # Olayları kontrol et
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:  # Boşluk tuşuna basıldığında kuş zıplar
                    bird_velocity = jump_strength

        # Kuşun hareketini hesapla
        bird_velocity += gravity
        bird_y += bird_velocity

        # Boruları ekle ve hareket ettir
        if len(pipes) == 0 or pipes[-1]['top_rect'].x < screen_width - 200:
            add_pipe()
        pipes = move_pipes()

        # Çarpışma kontrolü
        if check_collision():
            restart_button = game_over_message()

            # Yeniden başlama butonuna tıklama kontrolü
            restart = False
            while not restart:
                for event in pygame.event.get():
                    if event.type == pygame.QUIT:
                        running = False
                        restart = True
                    if event.type == pygame.MOUSEBUTTONDOWN:
                        if restart_button.is_clicked(event.pos):
                            restart_game()
                            restart = True

        # Kuşu ve boruları çiz
        draw_bird(bird_y)
        draw_pipes()

        # Skoru ekrana yazdır
        score_text = font.render(f"Score: {score}", True, BLACK)
        screen.blit(score_text, (10, 10))

        pygame.display.update()
        clock.tick(60)  # FPS (Frames Per Second)

    pygame.quit()

# Oyun başlatma
game_loop()
