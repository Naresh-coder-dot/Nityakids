# Nityakids
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.textinput import TextInput
from kivy.uix.screenmanager import Screen, ScreenManager
from kivy.animation import Animation
from kivy.core.audio import SoundLoader
from kivy.clock import Clock
from kivy.uix.image import Image
import random

# Sound effects (add sound files to the same directory as this script)
correct_sound = SoundLoader.load('correct.wav')  # Add a correct sound file
incorrect_sound = SoundLoader.load('incorrect.wav')  # Add an incorrect sound file

# Main Menu Screen
class MainMenuScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        layout = BoxLayout(orientation='vertical', padding=50, spacing=20)

        # Game title
        title = Label(text="Nitya Kids", font_size=50, color=(0, 0.5, 1, 1))

        # Start game button
        start_button = Button(text="Start Game", background_color=(0, 0.7, 0, 1), font_size=30, size_hint_y=None, height=100)
        start_button.bind(on_press=self.start_game)

        # Add widgets to layout
        layout.add_widget(title)
        layout.add_widget(start_button)
        self.add_widget(layout)

    def start_game(self, instance):
        self.manager.current = 'game'  # Switch to the game screen

# Game Screen
class GameScreen(Screen):
    def __init__(self, **kwargs):
        super().__init__(**kwargs)
        self.score = 0
        self.question = ""
        self.answer = ""
        self.user_text = ""

        # Main layout
        self.layout = BoxLayout(orientation='vertical', padding=50, spacing=20)

        # Score label
        self.score_label = Label(text=f"Score: {self.score}", font_size=30, color=(0, 0, 0, 1))

        # Question label
        self.question_label = Label(text="", font_size=40, color=(0, 0, 0, 1))

        # Input box for user's answer
        self.input_box = TextInput(hint_text="Your Answer", multiline=False, font_size=30, size_hint_y=None, height=60)
        self.input_box.bind(on_text_validate=self.check_answer)

        # Feedback label (correct/incorrect messages)
        self.feedback_label = Label(text="", font_size=30, color=(0, 0, 0, 1))

        # Add widgets to layout
        self.layout.add_widget(self.score_label)
        self.layout.add_widget(self.question_label)
        self.layout.add_widget(self.input_box)
        self.layout.add_widget(self.feedback_label)
        self.add_widget(self.layout)

        # Generate the first question
        self.generate_question()

    def generate_question(self):
        """Generate a random math question."""
        num1 = random.randint(1, 10)
        num2 = random.randint(1, 10)
        operator = random.choice(['+', '-', '*'])
        self.question = f"{num1} {operator} {num2}"
        self.answer = str(eval(self.question))
        self.question_label.text = self.question

    def check_answer(self, instance):
        """Check if the user's answer is correct."""
        if self.input_box.text == self.answer:
            self.score += 1
            self.feedback_label.text = "Correct! Great job!"
            self.feedback_label.color = (0, 1, 0, 1)  # Green
            if correct_sound:
                correct_sound.play()
        else:
            self.feedback_label.text = f"Incorrect. The answer was {self.answer}."
            self.feedback_label.color = (1, 0, 0, 1)  # Red
            if incorrect_sound:
                incorrect_sound.play()

        # Update score and clear input box
        self.score_label.text = f"Score: {self.score}"
        self.input_box.text = ""

        # Schedule the next question after 2 seconds
        Clock.schedule_once(self.next_question, 2)

    def next_question(self, dt):
        """Generate the next question and clear feedback."""
        self.generate_question()
        self.feedback_label.text = ""

# Screen Manager
class ScreenManagerApp(App):
    def build(self):
        sm = ScreenManager()
        sm.add_widget(MainMenuScreen(name='menu'))  # Main menu screen
        sm.add_widget(GameScreen(name='game'))     # Game screen
        return sm

# Run the App
if __name__ == "__main__":
    ScreenManagerApp().run()
