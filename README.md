# Creation-of-Colors
In this simulation, I'm exploring the RGB color model, where colors are created by adjusting the intensities of Red, Green, and Blue. By mixing these colors, I can create new ones, like Yellow, Cyan, or White. It helps me understand how colors blend and how digital screens use this model to display images.
import pygame
import sys

# Initialize pygame
pygame.init()

# Constants for the screen dimensions and colors
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 600
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)
WHITE = (255, 255, 255)
GRAY = (200, 200, 200)
BLACK = (0, 0, 0)

# Set up the display
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Color Mixing Simulation")

# Colors of the circles
circles = [
    {"color": RED, "pos": (200, 200), "radius": 100, "dragging": False},
    {"color": GREEN, "pos": (400, 200), "radius": 100, "dragging": False},
    {"color": BLUE, "pos": (300, 350), "radius": 100, "dragging": False},
]

# Sliders for adjusting the intensity of each color
sliders = {
    "red": {"value": 255, "y": 50, "color": RED},
    "green": {"value": 255, "y": 100, "color": GREEN},
    "blue": {"value": 255, "y": 150, "color": BLUE},
}

# Function to blend colors
def blend_colors(c1, c2, c3=None, alpha=0.5):
    """
    Blends two or three colors based on the alpha blending technique.
    """
    if c3 is not None:
        # Blending three colors
        r = int((c1[0] + c2[0] + c3[0]) * alpha / 3)
        g = int((c1[1] + c2[1] + c3[1]) * alpha / 3)
        b = int((c1[2] + c2[2] + c3[2]) * alpha / 3)

        # Ensure pure white (255, 255, 255) appears only when all colors are fully overlapped
        if r == 255 and g == 255 and b == 255:
            return WHITE
    else:
        # Blending two colors
        r = int(c1[0] * (1 - alpha) + c2[0] * alpha)
        g = int(c1[1] * (1 - alpha) + c2[1] * alpha)
        b = int(c1[2] * (1 - alpha) + c2[2] * alpha)
    
    return (r, g, b)

# Function to draw the circles and sliders
def draw_scene():
    # Draw the circles
    for circle in circles:
        pygame.draw.circle(screen, circle["color"], circle["pos"], circle["radius"])

    # Draw the sliders
    for slider in sliders.values():
        pygame.draw.rect(screen, GRAY, (50, slider["y"], 300, 10))  # Slider background
        pygame.draw.rect(screen, slider["color"], (50 + slider["value"], slider["y"] - 5, 10, 20))  # Slider handle

# Function to check if the mouse is within a circle
def is_mouse_over_circle(circle, mouse_pos):
    dx = mouse_pos[0] - circle["pos"][0]
    dy = mouse_pos[1] - circle["pos"][1]
    distance = (dx**2 + dy**2)**0.5
    return distance < circle["radius"]

# Function to create and display the intro screen
def display_intro_screen():
    screen.fill(BLACK)  # Fill the screen with black
    font = pygame.font.SysFont('Arial', 24)
    
    title = font.render("Welcome to the Color Mixing Simulation!", True, WHITE)
    description = font.render(
        "In this simulation, you will explore how RGB primary colors mix to create new colors.",
        True, WHITE
    )
    instructions = font.render(
        "Use the sliders to adjust color intensity and move the circles to mix colors in real time.",
        True, WHITE
    )
    more_instructions = font.render(
        "Press 'Let's Begin' to start the simulation.", True, WHITE
    )

    # Draw the text
    screen.blit(title, (50, 50))
    screen.blit(description, (50, 100))
    screen.blit(instructions, (50, 150))
    screen.blit(more_instructions, (50, 200))

    # Draw the "Let's Begin" button
    pygame.draw.rect(screen, GRAY, (SCREEN_WIDTH // 2 - 100, SCREEN_HEIGHT - 150, 200, 50))
    start_button_text = font.render("Let's Begin", True, WHITE)
    screen.blit(start_button_text, (SCREEN_WIDTH // 2 - 75, SCREEN_HEIGHT - 140))

    pygame.display.flip()

# Main game loop
dragging_slider = False
dragging_circle = None
intro_screen = True  # Flag to determine which screen to show
screen_rect = pygame.Rect(0, 0, SCREEN_WIDTH, SCREEN_HEIGHT)

while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()
        
        # If we're on the intro screen, handle the "Let's Begin" button
        if intro_screen:
            if event.type == pygame.MOUSEBUTTONDOWN:
                mouse_pos = pygame.mouse.get_pos()
                # Check if the "Let's Begin" button is clicked
                if (SCREEN_WIDTH // 2 - 100 <= mouse_pos[0] <= SCREEN_WIDTH // 2 + 100 and
                    SCREEN_HEIGHT - 150 <= mouse_pos[1] <= SCREEN_HEIGHT - 100):
                    intro_screen = False  # Start the main simulation

        # Handle dragging of sliders or circles (only after intro screen is dismissed)
        if not intro_screen:
            # Handle mouse button down events (start dragging circles or sliders)
            if event.type == pygame.MOUSEBUTTONDOWN:
                mouse_pos = pygame.mouse.get_pos()
                # Check if a slider is being clicked
                for slider_key, slider in sliders.items():
                    if 50 <= mouse_pos[0] <= 350 and slider["y"] - 5 <= mouse_pos[1] <= slider["y"] + 15:
                        dragging_slider = slider_key
                        break
                # Check if a circle is being clicked
                if not dragging_slider:
                    for circle in circles:
                        if is_mouse_over_circle(circle, mouse_pos):
                            dragging_circle = circle
                            break

            # Handle mouse button up events (stop dragging)
            elif event.type == pygame.MOUSEBUTTONUP:
                dragging_slider = False
                dragging_circle = None
            
            # Handle mouse motion events (dragging)
            elif event.type == pygame.MOUSEMOTION:
                if dragging_slider:
                    slider = sliders[dragging_slider]
                    slider["value"] = max(0, min(255, mouse_pos[0] - 50))
                elif dragging_circle:
                    dragging_circle["pos"] = pygame.mouse.get_pos()

            # Update circle colors based on the slider values
            circles[0]["color"] = (sliders["red"]["value"], 0, 0)
            circles[1]["color"] = (0, sliders["green"]["value"], 0)
            circles[2]["color"] = (0, 0, sliders["blue"]["value"])

            # Determine the color based on the overlapping circles
            overlapping_color = BLACK  # Default color if no overlap
            overlap_count = 0

            # Calculate the overlap color for the first circle
            for i, circle1 in enumerate(circles):
                for j, circle2 in enumerate(circles):
                    if i < j:
                        # Calculate overlap based on the positions of the circles
                        distance = ((circle1["pos"][0] - circle2["pos"][0])**2 + 
                                    (circle1["pos"][1] - circle2["pos"][1])**2)**0.5

                        # Determine how much overlap there is
                        overlap = max(0, circle1["radius"] + circle2["radius"] - distance) / min(circle1["radius"], circle2["radius"])
                        
                        # Blend the colors based on the overlap
                        if overlap > 0.5:
                            overlapping_color = blend_colors(circle1["color"], circle2["color"])
                            overlap_count += 1

            # If all circles overlap, set the screen to white
            if overlap_count == 3:
                overlapping_color = WHITE

            # Display the blended color
            screen.fill(overlapping_color)

            # Draw the scene (circles, sliders, etc.)
            draw_scene()

            # Update the display
            pygame.display.flip()
        else:
            # Display the intro screen while the intro screen flag is True
            display_intro_screen()
