import tkinter as tk
from tkinter import messagebox

MONEY = 10
ROUNDS = 5

class Player: # speletajs
    def __init__(self, name, money):
        self.name = name
        self.money = money

    def add_money(self, amount): # pievieno naudu
        self.money += amount

class tree: # izveido speles koka klasi
    def __init__(self, player, depth, score=None):
        self.player = player
        self.depth = depth
        self.score = score
        self.children = []

    def add_child(self, child):
        self.children.append(child)


class gameofchoice:
    def __init__(self):
        self.player1 = Player("Player 1", MONEY)
        self.player2 = Player("Player 2 (AI)", MONEY)
        self.current_round = 0
        self.ai_started_game = False
        self.root = tk.Tk()
        self.root.title("Symbol Fortune")
        self.create_widgets()

    def create_widgets(self): # izveido logu, kurā redz speli
        self.text_area = tk.Text(self.root, wrap="word", height=10, width=40)
        self.text_area.pack(padx=10, pady=10)
        self.text_area.config(state="disabled")

        button_frame = tk.Frame(self.root)
        button_frame.pack(padx=10, pady=10)

        self.button_a = tk.Button(button_frame, text="A", command=lambda: self.process_symbols(self.player1, "A"))
        self.button_b = tk.Button(button_frame, text="B", command=lambda: self.process_symbols(self.player1, "B"))
        self.button_play_again = tk.Button(button_frame, text="Play Again", command=self.reset_game)
        self.button_play_again.config(state="disabled")
        self.button_a.pack(side="left", padx=5)
        self.button_b.pack(side="left", padx=5)
        self.button_play_again.pack(side="left", padx=5)

        start_frame = tk.Frame(self.root)
        start_frame.pack(padx=10, pady=10)

        self.button_player_start = tk.Button(start_frame, text="Player starts", command=self.player_starts)
        self.button_ai_start = tk.Button(start_frame, text="AI starts", command=self.ai_starts)
        self.button_player_start.pack(side="left", padx=5)
        self.button_ai_start.pack(side="left", padx=5)

    def append_text(self, text): # darbošanās ar tekstu
        self.text_area.config(state="normal")
        self.text_area.insert("end", text + "\n")
        self.text_area.see("end")
        self.text_area.config(state="disabled")

    def start(self):
        self.root.mainloop()

    def process_symbols(self, player, symbol): # izveido pašu spēles procesu
        score = 1 if symbol == "A" else -2 # ja izvēlēsies A, tad pievienosies 1 punkts, ja B, tad atņems 2 punktus
        player.add_money(score)
        self.append_text(f"{player.name} chose symbol {symbol}")
        self.append_text(f"{player.name}'s score: {player.money}")

        if self.current_round < ROUNDS - 1:# ja nav pēdējais gājiens, tad spēle turpinās
            self.play_round()
        else:
            if not self.ai_started_game:
                self.play_round(last_round=True)
            else:
                self.display_results()

        self.current_round += 1

    def play_round(self, last_round=False):
        self.button_a.config(state="disabled")
        self.button_b.config(state="disabled")

        ai_symbol = self.minimax_choice(self.player2, self.player1, 3)
        ai_score = 1 if ai_symbol == "A" else -2
        self.player2.add_money(ai_score)
        self.append_text(f"{self.player2.name} chose symbol {ai_symbol}")
        self.append_text(f"{self.player2.name}'s score: {self.player2.money}")

        if not last_round: # ja nav pēdējais gājiens, tad spēle turpinās
            self.button_a.config(state="normal")
            self.button_b.config(state="normal")
        else:
            self.display_results()

    def minimax(self, node, maximizing_player): # minimax algoritms
        if not node.children:
            return node.score

        if maximizing_player:
            max_eval = float('-inf')
            for child in node.children:
                eval = self.minimax(child, False)
                max_eval = max(max_eval, eval)
            return max_eval
        else:
            min_eval = float('inf')
            for child in node.children:
                eval = self.minimax(child, True)
                min_eval = min(min_eval, eval)
            return min_eval

    def generate_game_tree(self, player1, player2, depth): # ģenerē spēles koku
        if depth == 0:
            return tree(player1, depth, player1.money - player2.money)

        root = tree(player1, depth)
        for symbol in ["A", "B"]:
            score = 1 if symbol == "A" else -2
            player1.add_money(score)
            child = self.generate_game_tree(player2, player1, depth - 1)
            root.add_child(child)
            player1.add_money(-score)

        return root

    def minimax_choice(self, ai_player, human_player, depth):
        root = self.generate_game_tree(ai_player, human_player, depth)
        best_move = None
        best_value = float('-inf')

        for child in root.children:
            value = self.minimax(child, False)
            if value > best_value:
                best_value = value
                best_move = child.player

        return "A" if best_move.money >= ai_player.money + 1 else "B" # izvēlas labāko spēles gājienu

    def player_starts(self):
        self.button_player_start.config(state="disabled")
        self.button_ai_start.config(state="disabled")
        self.button_a.config(state="normal")
        self.button_b.config(state="normal")

    def ai_starts(self):
        self.button_player_start.config(state="disabled")
        self.button_ai_start.config(state="disabled")
        self.ai_started_game = True
        self.play_round()

    def display_results(self):
        getwinner = self.player1.name if self.player1.money > self.player2.money else self.player2.name
        messagebox.showinfo("Game Over",
                            f"Final scores:\n{self.player1.name}: {self.player1.money}\n{self.player2.name}: {self.player2.money}\n\nWinner: {getwinner}")
        self.button_play_again.config(state="normal")

    def reset_game(self):
        self.player1.money = MONEY
        self.player2.money = MONEY
        self.current_round = 0
        self.ai_started_game = False
        self.text_area.config(state="normal")
        self.text_area.delete(1.0, "end")
        self.text_area.config(state="disabled")
        self.button_play_again.config(state="disabled")
        self.button_player_start.config(state="normal")
        self.button_ai_start.config(state="normal")

    def start(self):
        self.root.mainloop()

def main():
    game = gameofchoice()
    game.start()

if __name__ == "__main__":
    main()
