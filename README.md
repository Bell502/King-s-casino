pip install kivy
from kivy.app import App
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.button import Button
from kivy.uix.label import Label
from kivy.uix.image import Image
from kivy.uix.textinput import TextInput
from kivy.core.window import Window
from kivy.core.audio import SoundLoader
import random

class MainApp(App):
    def __init__(self, **kwargs):
        super(MainApp, self).__init__(**kwargs)
        self.balance = 1000000  # 초기 가상 머니 설정
        self.current_game = None  # 현재 게임 설정
        self.sounds = {
            "win": SoundLoader.load('win_sound.mp3'),
            "lose": SoundLoader.load('lose_sound.mp3'),
            "draw": SoundLoader.load('draw_sound.mp3'),
            "slot_win": SoundLoader.load('slot_win_sound.mp3'),
            "jackpot": SoundLoader.load('jackpot_sound.mp3'),
        }

    def build(self):
        Window.clearcolor = (1, 1, 1, 1)  # 배경색 흰색
        layout = BoxLayout(orientation='vertical', padding=20, spacing=10)

        # 마스코트 이미지 추가
        mascot = Image(source='golden_tiger.png', size_hint=(1, 0.3))  # 황금호랑이 이미지
        layout.add_widget(mascot)

        self.balance_label = Label(text=f"Current Balance: ${self.balance}", font_size='24sp')
        layout.add_widget(self.balance_label)

        layout.add_widget(Label(text="Select a game:", font_size='24sp'))

        # 각 게임 버튼 추가
        games = [
            ("Baccarat", self.baccarat),
            ("Powerball", self.powerball),
            ("Slot Machine", self.slot_machine),
            ("Toto", self.toto),
            ("Roulette", self.roulette),
            ("Virtual Horse Racing", self.virtual_horse_racing),
            ("Texas Hold'em", self.holdem),
            ("Matgo", self.matgo),
            ("Blackjack", self.blackjack),
        ]

        for game in games:
            btn = Button(text=game[0], size_hint=(1, 0.1))
            btn.bind(on_press=lambda instance, g=game[1]: self.start_game(g))
            layout.add_widget(btn)

        return layout

    def start_game(self, game_func):
        self.current_game = game_func
        self.show_bet_prompt()  # 베팅 금액 입력 프롬프트 표시

    def show_bet_prompt(self):
        bet_label = Label(text="Enter your bet:", font_size='20sp')
        bet_input = TextInput(multiline=False, input_filter='int', size_hint=(1, 0.1))
        bet_button = Button(text="Place Bet", size_hint=(1, 0.1))

        # 베팅 버튼 클릭 시 처리
        def place_bet(instance):
            try:
                bet_amount = int(bet_input.text)
                if 0 < bet_amount <= self.balance:
                    self.balance -= bet_amount
                    self.balance_label.text = f"Current Balance: ${self.balance}"
                    self.current_game(bet_amount)  # 게임 시작
                    self.clear_bet_prompt(bet_label, bet_input, bet_button)
                else:
                    bet_label.text = "Invalid bet amount. Please try again."
            except ValueError:
                bet_label.text = "Please enter a valid number."

        bet_button.bind(on_press=place_bet)

        self.root.clear_widgets()
        self.root.add_widget(bet_label)
        self.root.add_widget(bet_input)
        self.root.add_widget(bet_button)

    def clear_bet_prompt(self, bet_label, bet_input, bet_button):
        self.root.clear_widgets()  # 입력 프롬프트 제거
        self.build()  # 메뉴로 돌아가기

    def baccarat(self, bet_amount):
        player_score = random.randint(0, 9)
        banker_score = random.randint(0, 9)
        result = f"Player's score: {player_score}\nBanker's score: {banker_score}\n"
        if player_score > banker_score:
            result += f"Player wins! You won ${bet_amount}!"
            self.balance += bet_amount * 2  # 베팅 금액의 2배 지급
            self.sounds["win"].play()  # 승리 사운드 재생
        elif banker_score > player_score:
            result += "Banker wins! You lost your bet."
            self.sounds["lose"].play()  # 패배 사운드 재생
        else:
            result += "It's a tie! You get your bet back."
            self.balance += bet_amount  # 베팅 금액 반환
            self.sounds["draw"].play()  # 무승부 사운드 재생

        self.show_result(result)

    def powerball(self, bet_amount):
        white_balls = random.sample(range(1, 70), 5)
        power_ball = random.randint(1, 26)
        result = f"Your Powerball numbers are: {sorted(white_balls)} and Powerball: {power_ball}"
        result += "\nYou can play again!"
        self.show_result(result)

    def slot_machine(self, bet_amount):
        symbols = ["🍒", "🍋", "🍊", "🍉", "⭐", "💎"]
        spin_result = [random.choice(symbols) for _ in range(3)]
        result = f"Spin result: {' '.join(spin_result)}"
        if len(set(spin_result)) == 1:
            winnings = bet_amount * 5  # 5배 지급
            result += f"\nJackpot! You win ${winnings}!"
            self.balance += winnings
            self.sounds["jackpot"].play()  # 잭팟 사운드 재생
        else:
            result += "\nTry again! You lost your bet."
            self.sounds["lose"].play()  # 패배 사운드 재생

        self.show_result(result)

    def toto(self, bet_amount):
        numbers = random.sample(range(1, 49), 6)
        result = f"Your Lotto numbers are: {sorted(numbers)}"
        result += "\nYou can play again!"
        self.show_result(result)

    def roulette(self, bet_amount):
        spin_result = random.randint(0, 36)
        result = f"The roulette spin resulted in: {spin_result}"
        # 예를 들어 홀수 또는 짝수에 베팅했다면:
        if spin_result % 2 == 0:
            winnings = bet_amount * 2  # 2배 지급
            result += f"\nYou win ${winnings}!"
            self.balance += winnings
            self.sounds["win"].play()  # 승리 사운드 재생
        else:
            result += "\nYou lost your bet."
            self.sounds["lose"].play()  # 패배 사운드 재생

        self.show_result(result)

    def virtual_horse_racing(self, bet_amount):
        horses = ['Horse 1', 'Horse 2', 'Horse 3', 'Horse 4']
        winner = random.choice(horses)
        result = f"The winner is: {winner}"
        if winner == 'Horse 1':  # 특정 말에 베팅한 경우
            winnings = bet_amount * 3  # 3배 지급
            result += f"\nYou win ${winnings}!"
            self.balance += winnings
            self.sounds["win"].play()  # 승리 사운드 재생
        else:
            result += "\nYou lost your bet."
            self.sounds["lose"].play()  # 패배 사운드 재생

        self.show_result(result)

    def holdem(self, bet_amount):
        deck = [f'{rank} of {suit}' for suit in ['Hearts', 'Diamonds', 'Clubs', 'Spades']
                for rank in ['2', '3', '4', '5', '6', '7', '8', '9', '10', 'Jack', 'Queen', 'King', 'Ace']]
        random.shuffle(deck)
        player_hand = [deck.pop(), deck.pop()]
        result = f"Your hand: {player_hand}"
        self.show_result(result)

    def matgo(self, bet_amount):
        player_hand = [random.randint(1, 6), random.randint(1, 6)]
        result = f"Your cards: {player_hand}"
        self.show_result(result)

    def calculate_hand_value(self, hand):
        value = 0
        aces = 0
        for card in hand:
            rank = card[0]
            if rank in ['Jack', 'Queen', 'King']:
                value += 10
            elif rank == 'Ace':
                aces += 1
                value += 11
            else:
                value += int(rank)
        while value > 21 and aces:
            value -= 10
            aces -= 1
        return value

    def blackjack(self, bet_amount):
        deck = [f'{rank} of {suit}' for suit in ['Hearts', 'Diamonds', 'Clubs', 'Spades']
                for rank in ['2','3', '4', '5', '6', '7', '8', '9', '10', 'Jack', 'Queen', 'King', 'Ace']]
        random.shuffle(deck)

        player_hand = [deck.pop(), deck.pop()]
        dealer_hand = [deck.pop(), deck.pop()]

        player_value = self.calculate_hand_value(player_hand)
        dealer_value = self.calculate_hand_value(dealer_hand)

        result = f"Your hand: {player_hand} (Value: {player_value})\n"
        result += f"Dealer's hand: {dealer_hand[0]} and Hidden Card\n"  # 딜러의 첫 번째 카드만 보여줌

        # 플레이어가 추가 카드를 원한다고 가정
        while player_value < 21:
            result += "Hit or Stand? (h/s): "
            # 이 부분을 수정하여 UI에서 입력받는 로직을 추가해야 합니다.
            # 예시로 'h'를 사용해 임의로 히트하는 로직을 추가합니다.
            player_hand.append(deck.pop())
            player_value = self.calculate_hand_value(player_hand)
            result += f"New hand: {player_hand} (Value: {player_value})\n"
            if player_value > 21:
                result += "Bust! You lose."
                self.sounds["lose"].play()  # 패배 사운드 재생
                self.balance -= bet_amount  # 베팅 금액 차감
                self.show_result(result)
                return

        # 딜러가 카드를 공개하고 규칙에 따라 카드를 뽑는다.
        result += f"Dealer's hand: {dealer_hand} (Value: {dealer_value})\n"
        while dealer_value < 17:
            dealer_hand.append(deck.pop())
            dealer_value = self.calculate_hand_value(dealer_hand)
            result += f"Dealer draws a card: {dealer_hand} (Value: {dealer_value})\n"

        # 결과 판단
        if dealer_value > 21 or player_value > dealer_value:
            winnings = bet_amount * 2  # 2배 지급
            result += f"You win! You earned ${winnings}!"
            self.balance += winnings
            self.sounds["win"].play()  # 승리 사운드 재생
        elif player_value < dealer_value:
            result += "You lose your bet."
            self.sounds["lose"].play()  # 패배 사운드 재생
        else:
            result += "It's a tie! You get your bet back."
            self.balance += bet_amount  # 베팅 금액 반환
            self.sounds["draw"].play()  # 무승부 사운드 재생

        self.show_result(result)

    def show_result(self, result):
        self.root.clear_widgets()
        result_label = Label(text=result, font_size='20sp')
        back_button = Button(text="Back to Main Menu", size_hint=(1, 0.1))
        back_button.bind(on_press=self.build)
        self.root.add_widget(result_label)
        self.root.add_widget(back_button)

if __name__ == "__main__":
    MainApp().run()
