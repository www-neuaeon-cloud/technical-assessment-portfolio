import requests
from bs4 import BeautifulSoup
import re

def decode_secret_message(url: str):
    """
    Retrieves data from a Google Doc URL, parses character and coordinate data,
    and prints a 2D grid that reveals a secret message.

    Args:
        url: The public URL of the Google Doc containing the data.
    """
    try:
        response = requests.get(url, timeout=10)
        # Raise an HTTPError for bad responses (4xx or 5xx)
        response.raise_for_status()
    except requests.exceptions.RequestException as e:
        print(f"Error: Could not retrieve data from URL. {e}")
        return

    # Use BeautifulSoup to parse the HTML content of the document
    soup = BeautifulSoup(response.text, 'html.parser')
    
    # Extract the text from all paragraph tags, which contain the data lines
    lines = [p.get_text() for p in soup.find_all('p')]

    points = []
    max_x = 0
    max_y = 0

    # Regex to find lines with: 'U+HEX_CODE', x_coord, y_coord
    # This handles potential whitespace variations
    pattern = re.compile(r"'U\+([0-9A-F]+)'\s*,\s*(\d+)\s*,\s*(\d+)")

    for line in lines:
        match = pattern.search(line)
        if match:
            hex_code, x_str, y_str = match.groups()
            
            # Convert extracted parts to their correct types
            character = chr(int(hex_code, 16))
            x = int(x_str)
            y = int(y_str)

            points.append({'char': character, 'x': x, 'y': y})

            # Track grid dimensions
            if x > max_x:
                max_x = x
            if y > max_y:
                max_y = y

    if not points:
        print("No valid character data found in the document.")
        return

    # Create a 2D grid initialized with spaces
    grid_width = max_x + 1
    grid_height = max_y + 1
    grid = [[' ' for _ in range(grid_width)] for _ in range(grid_height)]

    # Populate the grid with characters at their (x, y) coordinates
    for point in points:
        # Array indices are [row][col], which corresponds to [y][x]
        grid[point['y']][point['x']] = point['char']

    # Print the final grid row by row
    for row in grid:
        print("".join(row))
