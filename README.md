import random
import tkinter as tk
import requests

# Initialize global variables
word_list = ["stupendous", "amazing", "brilliant", "fantastic", "marvelous"]
guessed_letters = set()
definition = ""
attempts = 6
generated_word = ""
structure_word = []

# ASCII Art for Hangman stages
stages = [
    """
       ------
       |    |
       |
       |
       |
       -
    """,
    """
       ------
       |    |
       |    O
       |
       |
       -
    """,
    """
       ------
       |    |
       |    O
       |    |
       |
       -
    """,
    """
       ------
       |    |
       |    O
       |   /|
       |
       -
    """,
    """
       ------
       |    |
       |    O
       |   /|\\
       |
       -
    """,
    """
       ------
       |    |
       |    O
       |   /|\\
       |   /
       -
    """,
    """
       ------
       |    |
       |    O
       |   /|\\
       |   / \\
       -
    """
]

# Function to fetch word definition from API
def fetch_definition(word):
    response = requests.get(f'https://api.dictionaryapi.dev/api/v2/entries/en/{word}')
    if response.status_code == 200:
        data = response.json()
        if data and "meanings" in data[0]:
            return data[0]["meanings"][0].get("definitions", [{}])[0].get("definition", "Definition not available.")
    return "Definition not found."

# Function to initialize/reset the game
def initialize_game():
    global generated_word, structure_word, attempts, guessed_letters, definition

    generated_word = random.choice(word_list)
    structure_word = ["_"] * len(generated_word)
    attempts = 6
    guessed_letters.clear()
    definition = fetch_definition(generated_word)

    word_label.config(text=" ".join(structure_word))
    hangman_label.config(text=stages[0])
    definition_label.config(text=f"Definition: {definition}")
    result_label.config(text="")
    entry.config(state=tk.NORMAL)
    guess_button.config(state=tk.NORMAL)

# Function to update the Hangman drawing
def update_hangman():
    hangman_label.config(text=stages[6 - attempts])

# Function to animate Hangman dance after winning
def hangman_dance(count=0):
    dance_moves = [
    '''
       O/
      /|  
      / \\ 
    ''',
    '''
       O
      /|\\
      / \\ 
    ''',
    '''
      \\O
       |\\
      / \\ 
    '''
    ]
    
    if count < 10:  
        hangman_label.config(text=dance_moves[count % len(dance_moves)])
        root.after(500, lambda: hangman_dance(count + 1))
    else:
        hangman_label.config(text=dance_moves[1])  # Set back to center position

# Function to check user's guess
def check_guess():
    global attempts
    entered_letter = entry.get().lower()

    if len(entered_letter) != 1 or not entered_letter.isalpha():
        result_label.config(text="❌ Please enter a valid single letter.")
        return

    if entered_letter in guessed_letters:
        result_label.config(text="⚠️ You've already guessed that letter!")
        return

    entry.delete(0, tk.END)  # Now deleting after checking validation

    guessed_letters.add(entered_letter)
    structure_word[:] = [entered_letter if generated_word[i] == entered_letter else structure_word[i] for i in range(len(generated_word))]
    word_label.config(text=" ".join(structure_word))

    if entered_letter not in generated_word:
        attempts -= 1
        update_hangman()
        result_label.config(text=f"❌ Wrong guess! Remaining attempts: {attempts}")

    if "_" not in structure_word:  # Player wins
        result_label.config(text="🎉 Congratulations! You freed the Hangman!")
        entry.config(state=tk.DISABLED)
        guess_button.config(state=tk.DISABLED)
        hangman_dance()  # Start the dance animation
    elif attempts == 0:  # Player loses
        result_label.config(text=f"😢 You lost! The word was: {generated_word}")
        entry.config(state=tk.DISABLED)
        guess_button.config(state=tk.DISABLED)

# Function to reset game (calls initialize_game)
def reset_game():
    initialize_game()

# Tkinter UI Setup
root = tk.Tk()
root.title("Hangman Game")

center_frame = tk.Frame(root)
center_frame.pack(side=tk.LEFT, padx=50, pady=20)
right_frame = tk.Frame(root)
right_frame.pack(side=tk.RIGHT, padx=50, pady=20)

hangman_label = tk.Label(right_frame, text=stages[0], font=("Courier", 14), justify=tk.LEFT)
hangman_label.pack()

word_label = tk.Label(center_frame, text="", font=("Helvetica", 18, "bold"))
word_label.pack()

definition_label = tk.Label(center_frame, text="", font=("Helvetica", 10), wraplength=300, justify="left")
definition_label.pack(pady=10)

input_title = tk.Label(center_frame, text="Enter a letter:", font=("Helvetica", 12))
input_title.pack(pady=5)

entry = tk.Entry(center_frame, font=("Helvetica", 14))
entry.pack(pady=10)

guess_button = tk.Button(center_frame, text="Guess", font=("Helvetica", 14), command=check_guess)
guess_button.pack()

reset_button = tk.Button(center_frame, text="Reset", font=("Helvetica", 14), command=reset_game)
reset_button.pack(pady=10)

result_label = tk.Label(center_frame, text="", font=("Helvetica", 12))
result_label.pack(pady=10)

# Now calling initialize_game() after creating the UI elements
initialize_game()

root.mainloop()
