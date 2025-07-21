# Quiz-game-import json
import random
import time
import os
from collections import defaultdict
from colorama import Fore, Back, Style, init

# Initialize colorama
init(autoreset=True)

class QuizGame:
    def __init__(self):
        self.questions = self.load_questions()
        self.categories = list(self.questions.keys())
        self.session_stats = defaultdict(lambda: {"correct": 0, "total": 0})
        self.current_score = 0
        self.username = ""
        
    def load_questions(self):
        """Load questions from questions.json"""
        try:
            with open('questions.json', 'r') as file:
                return json.load(file)
        except FileNotFoundError:
            print(f"{Fore.RED}Error: 'questions.json' not found. Using default questions.")
            return {
                "Science": {
                    "Easy": [
                        {"question": "What is the chemical symbol for gold?", "options": ["Au", "Ag", "Fe", "Hg"], "answer": "Au"},
                        {"question": "Which planet is known as the Red Planet?", "options": ["Earth", "Mars", "Venus", "Jupiter"], "answer": "Mars"}
                    ],
                    "Medium": [
                        {"question": "What does DNA stand for?", "options": ["Dynamic Nuclear Acid", "Deoxyribonucleic Acid", "Dual Nucleotide Assembly", "Dinitrogen Acid"], "answer": "Deoxyribonucleic Acid"}
                    ],
                    "Hard": [
                        {"question": "Which element has the highest melting point?", "options": ["Tungsten", "Iron", "Gold", "Carbon"], "answer": "Tungsten"}
                    ]
                },
                "History": {
                    "Easy": [
                        {"question": "Who was the first president of the USA?", "options": ["Abraham Lincoln", "George Washington", "Thomas Jefferson", "John Adams"], "answer": "George Washington"}
                    ],
                    "Hard": [
                        {"question": "What year did World War II end?", "options": ["1943", "1945", "1950", "1939"], "answer": "1945"}
                    ]
                }
            }

    def clear_screen(self):
        """Clear terminal screen"""
        os.system('cls' if os.name == 'nt' else 'clear')

    def print_header(self, text):
        """Print styled header"""
        print(f"\n{Fore.CYAN}{'='*50}")
        print(f"{text.center(50)}")
        print(f"{'='*50}{Style.RESET_ALL}")

    def display_welcome(self):
        """Show welcome message"""
        self.clear_screen()
        print(f"\n{Fore.MAGENTA}{Back.WHITE}{' QUIZ GAME MASTER '.center(50, '★')}{Style.RESET_ALL}")
        print(f"{Fore.GREEN}\nWelcome to the ultimate quiz challenge!\n")
        print(f"{Fore.YELLOW}Test your knowledge across various categories.")
        print(f"You'll earn points for each correct answer.\n")

    def get_username(self):
        """Get player name"""
        self.username = input(f"{Fore.CYAN}Enter your name: {Style.RESET_ALL}").strip()
        if not self.username:
            self.username = "Player"

    def select_category(self):
        """Let user select a category"""
        self.print_header("SELECT CATEGORY")
        for idx, category in enumerate(self.categories, 1):
            print(f"{Fore.YELLOW}{idx}. {category}")
        
        while True:
            try:
                choice = int(input(f"\n{Fore.CYAN}Choose category (1-{len(self.categories)}): ")) - 1
                if 0 <= choice < len(self.categories):
                    return self.categories[choice]
                print(f"{Fore.RED}Invalid choice!")
            except ValueError:
                print(f"{Fore.RED}Enter a number!")

    def select_difficulty(self, category):
        """Let user select difficulty level"""
        difficulties = list(self.questions[category].keys())
        self.print_header(f"SELECT DIFFICULTY ({category})")
        
        for idx, diff in enumerate(difficulties, 1):
            print(f"{Fore.YELLOW}{idx}. {diff}")
        
        while True:
            try:
                choice = int(input(f"\n{Fore.CYAN}Choose difficulty (1-{len(difficulties)}): ")) - 1
                if 0 <= choice < len(difficulties):
                    return difficulties[choice]
                print(f"{Fore.RED}Invalid choice!")
            except ValueError:
                print(f"{Fore.RED}Enter a number!")

    def ask_question(self, question_data):
        """Display question and get answer"""
        self.clear_screen()
        print(f"\n{Fore.BLUE}Question:{Style.RESET_ALL} {question_data['question']}")
        print(f"{Fore.BLUE}Options:{Style.RESET_ALL}")
        
        for idx, option in enumerate(question_data["options"], 1):
            print(f"{Fore.YELLOW}{idx}. {option}{Style.RESET_ALL}")
        
        while True:
            try:
                answer = input("\nYour answer (1-4): ")
                if not answer.isdigit():
                    raise ValueError
                
                answer_idx = int(answer) - 1
                if 0 <= answer_idx < len(question_data["options"]):
                    return question_data["options"][answer_idx]
                print(f"{Fore.RED}Choose between 1-{len(question_data['options'])}!")
            except ValueError:
                print(f"{Fore.RED}Please enter a valid number!")

    def run_quiz(self, category, difficulty):
        """Run a quiz session"""
        questions = random.sample(
            self.questions[category][difficulty],
            min(5, len(self.questions[category][difficulty]))
        )
        
        self.current_score = 0
        
        for idx, question in enumerate(questions, 1):
            user_answer = self.ask_question(question)
            correct_answer = question["answer"]
            
            if user_answer.lower() == correct_answer.lower():
                self.current_score += 1
                print(f"\n{Fore.GREEN}✔ Correct! (+1 point)")
            else:
                print(f"\n{Fore.RED}✘ Wrong! Correct answer: {correct_answer}")
            
            time.sleep(1.5)
        
        # Update session stats
        self.session_stats[f"{category}/{difficulty}"]["correct"] += self.current_score
        self.session_stats[f"{category}/{difficulty}"]["total"] += len(questions)
        
        # Show results
        self.clear_screen()
        self.print_header("QUIZ RESULTS")
        print(f"{Fore.GREEN}Category:{Style.RESET_ALL} {category}")
        print(f"{Fore.GREEN}Difficulty:{Style.RESET_ALL} {difficulty}")
        print(f"{Fore.GREEN}Score:{Style.RESET_ALL} {self.current_score}/{len(questions)}")
        
        percentage = (self.current_score / len(questions)) * 100
        performance = "Excellent! 🎯" if percentage >= 80 else "Good job! 👍" if percentage >= 50 else "Keep practicing! 📚"
        
        print(f"\n{Fore.MAGENTA}Performance:{Style.RESET_ALL} {performance} ({percentage:.0f}%)")
        
        input(f"\n{Fore.CYAN}Press Enter to continue...{Style.RESET_ALL}")

    def show_stats(self):
        """Display session statistics"""
        self.clear_screen()
        self.print_header("YOUR STATISTICS")
        
        print(f"{Fore.YELLOW}Player:{Style.RESET_ALL} {self.username}\n")
        
        for category_diff, stats in self.session_stats.items():
            category, diff = category_diff.split("/")
            percentage = (stats["correct"] / stats["total"]) * 100 if stats["total"] > 0 else 0
            
            print(f"{Fore.CYAN}{category} ({diff}){Style.RESET_ALL}:")
            print(f"  Correct: {stats['correct']}/{stats['total']}")
            print(f"  Accuracy: {percentage:.1f}%")
            print()

    def main_menu(self):
        """Display main menu"""
        while True:
            self.clear_screen()
            self.print_header("MAIN MENU")
            print(f"{Fore.GREEN}1.{Style.RESET_ALL} Start New Quiz")
            print(f"{Fore.GREEN}2.{Style.RESET_ALL} View Statistics")
            print(f"{Fore.GREEN}3.{Style.RESET_ALL} Exit")
            
            choice = input(f"\n{Fore.CYAN}Enter choice (1-3): {Style.RESET_ALL}")
            
            if choice == "1":
                category = self.select_category()
                difficulty = self.select_difficulty(category)
                self.run_quiz(category, difficulty)
            elif choice == "2":
                self.show_stats()
                input(f"\n{Fore.CYAN}Press Enter to continue...{Style.RESET_ALL}")
            elif choice == "3":
                print(f"\n{Fore.GREEN}Thanks for playing, {self.username}! Goodbye! 👋\n")
                break
            else:
                print(f"{Fore.RED}Invalid choice!")

    def run(self):
        """Run the quiz game"""
        self.clear_screen()
        self.display_welcome()
        self.get_username()
        self.main_menu()

if __name__ == "__main__":
    game = QuizGame()
    game.run()
{
    "Science": {
        "Easy": [
            {
                "question": "What is the chemical symbol for gold?",
                "options": ["Au", "Ag", "Fe", "Hg"],
                "answer": "Au"
            },
            {
                "question": "Which planet is known as the Red Planet?",
                "options": ["Venus", "Mars", "Jupiter", "Pluto"],
                "answer": "Mars"
            }
        ],
        "Hard": [
            {
                "question": "Which planet has the most moons?",
                "options": ["Saturn", "Jupiter", "Neptune", "Uranus"],
                "answer": "Saturn"
            }
        ]
    },
    "Movies": {
        "Easy": [
            {
                "question": "Who directed Titanic?",
                "options": ["Steven Spielberg", "James Cameron", "Peter Jackson", "George Lucas"],
                "answer": "James Cameron"
            }
        ],
        "Medium": [
            {
                "question": "Which actor played Neo in The Matrix?",
                "options": ["Brad Pitt", "Tom Cruise", "Keanu Reeves", "Leonardo DiCaprio"],
                "answer": "Keanu Reeves"
            }
        ]
    }
}
