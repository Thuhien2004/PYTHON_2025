# PYTHON_2025
Báo cáo bài tập lớn Python cuối học kỳ III - 2025
FILE: blackjack_gui_multi.py
---------------------------------------------------------------------------------------------------------------------------------------------------------------------

import tkinter as tk
from tkinter import messagebox, ttk
import random

RANKS = ["A", "2", "3", "4", "5", "6", "7", "8", "9", "10", "J", "Q", "K"]
SUITS = ["♠", "♥", "♦", "♣"]
CARD_VALUES = {"A": 11, "2": 2, "3": 3, "4": 4, "5": 5, "6": 6, "7": 7, "8": 8, "9": 9, "10": 10, "J": 10, "Q": 10, "K": 10}

class BJ_Card:
    def __init__(self, rank, suit):
        self.rank = rank
        self.suit = suit

    def value(self):
        return CARD_VALUES[self.rank]

    def __str__(self):
        return f"{self.rank}{self.suit}"

class BJ_Deck:
    def __init__(self):
        self.cards = [BJ_Card(rank, suit) for rank in RANKS for suit in SUITS]
        random.shuffle(self.cards)

    def draw(self):
        return self.cards.pop()

class BJ_Hand:
    def __init__(self):
        self.cards = []

    def add_card(self, card):
        self.cards.append(card)

    def score(self):
        total = sum(card.value() for card in self.cards)
        aces = sum(1 for card in self.cards if card.rank == "A")
        while total > 21 and aces:
            total -= 10
            aces -= 1
        return total

    def __str__(self):
        return ", ".join(str(card) for card in self.cards)

class BJ_Game:
    def __init__(self, num_players):
        self.deck = BJ_Deck()
        self.players = [BJ_Hand() for _ in range(num_players)]
        self.dealer = BJ_Hand()
        self.balances = [100] * num_players
        self.bets = [10] * num_players
        self.current = 0

    def deal_initial(self):
        for _ in range(2):
            for hand in self.players:
                hand.add_card(self.deck.draw())
            self.dealer.add_card(self.deck.draw())

    def hit_player(self):
        self.players[self.current].add_card(self.deck.draw())

    def player_bust(self):
        return self.players[self.current].score() > 21

    def dealer_play(self):
        while self.dealer.score() < 17:
            self.dealer.add_card(self.deck.draw())

    def settle(self):
        results = []
        dealer_score = self.dealer.score()
        for i, hand in enumerate(self.players):
            p_score = hand.score()
            bet = self.bets[i]
            if p_score > 21:
                results.append((i, "Thua"))
                self.balances[i] -= bet
            elif dealer_score > 21 or p_score > dealer_score:
                results.append((i, "Thắng"))
                self.balances[i] += bet
            elif p_score < dealer_score:
                results.append((i, "Thua"))
                self.balances[i] -= bet
            else:
                results.append((i, "Hòa"))
        return results

class BlackjackGUI:
    def __init__(self, root):
        self.root = root
        self.root.title("Blackjack GUI")
        self.root.configure(bg="#2c3e50")

        style = ttk.Style()
        style.configure("TButton", padding=6, relief="flat", background="#5aa4ef", foreground="black")

        self.start_frame = tk.Frame(root, bg="#7FAAD5")
        tk.Label(self.start_frame, text="Số người chơi: ", fg="white", bg="#2c3e50").pack(side=tk.LEFT)
        self.num_players_entry = tk.Entry(self.start_frame)
        self.num_players_entry.pack(side=tk.LEFT)
        tk.Button(self.start_frame, text="Bắt đầu", command=self.start_game, font=("Helvetica", 10, "bold"),
                    bg="#27ae60", fg="white").pack(side=tk.LEFT, padx=5)
        self.start_frame.pack(pady=10)

        self.labels = []
        self.control_frame = tk.Frame(root, bg="#2c3e50")
        self.hit_btn = tk.Button(self.control_frame, text="Hit", command=self.hit,
                        font=("Helvetica", 10, "bold"), bg="#2980b9", fg="white")
        self.stand_btn = tk.Button(self.control_frame, text="Stand", command=self.stand,
                        font=("Helvetica", 10, "bold"), bg="#e67e22", fg="white")
        self.restart_btn = tk.Button(self.control_frame, text="Restart", command=self.restart,
                            font=("Helvetica", 10, "bold"), bg="#c0392b", fg="white")

        for b in (self.hit_btn, self.stand_btn, self.restart_btn):
            b.pack(side=tk.LEFT, padx=5)

        self.dealer_label = tk.Label(root, fg="white", bg="#2c3e50", font=("Helvetica", 12, "bold"))

    def start_game(self):
        try:
            n = int(self.num_players_entry.get())
            if not (1 <= n <= 6): raise ValueError
        except ValueError:
            messagebox.showerror("Lỗi", "Chỉ được nhập 1-6 người chơi")
            return

        self.game = BJ_Game(n)
        self.game.deal_initial()
        self.labels = []
        self.start_frame.pack_forget()

        self.game_frame = tk.Frame(self.root, bg="#2c3e50")
        self.game_frame.pack()
        for i in range(n):
            lbl = tk.Label(self.game_frame, text="", fg="white", bg="#34495e")
            lbl.pack(anchor="w", padx=10)
            self.labels.append(lbl)

        self.dealer_label.pack(pady=10)
        self.control_frame.pack(pady=10)
        self.update_ui()

    def update_ui(self):
        for i, lbl in enumerate(self.labels):
            cards = str(self.game.players[i])
            score = self.game.players[i].score()
            money = self.game.balances[i]
            lbl.config(text=f"Người chơi {i+1}: {cards} | Điểm: {score} | Tiền: {money}$",
                       bg="yellow" if i == self.game.current else "#34495e", fg="black" if i == self.game.current else "white")

        self.dealer_label.config(text=f"Dealer: {str(self.game.dealer)} | Điểm: {self.game.dealer.score()}")

    def hit(self):
        self.game.hit_player()
        if self.game.player_bust():
            self.next_player()
        self.update_ui()

    def stand(self):
        self.next_player()
        self.update_ui()

    def next_player(self):
        if self.game.current + 1 < len(self.game.players):
            self.game.current += 1
        else:
            self.game.dealer_play()
            results = self.game.settle()
            for i, result in results:
                messagebox.showinfo(f"Người chơi {i+1}", f"Kết quả: {result}")
        self.update_ui()

    def restart(self):
        self.start_frame.pack()
        self.game_frame.pack_forget()
        self.control_frame.pack_forget()
        self.dealer_label.pack_forget()

if __name__ == "__main__":
    root = tk.Tk()
    app = BlackjackGUI(root)
    root.mainloop()
