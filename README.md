import discord
import random
from discord.ext import commands

# 디스코드 봇 생성
bot = commands.Bot(command_prefix='!')

# 사용자 점수 저장
user_scores = {}

# 게임 목록
games = {
    "블랙잭": "blackjack",
    "바카라": "baccarat",
    "토토": "toto",
    "슬롯": "slot_machine",
    "파워볼": "powerball",
}

def get_user_score(user):
    return user_scores.get(user.id, 100)  # 기본 점수 100으로 설정

def update_user_score(user, amount):
    user_scores[user.id] = get_user_score(user) + amount

# 1. 블랙잭 (Blackjack)
async def blackjack(ctx):
    # 블랙잭 로직
    user_score = get_user_score(ctx.author)
    bet_amount = await get_bet(ctx, user_score)

    if bet_amount is None:
        return  # 유효하지 않은 베팅인 경우

    user_cards = [random.choice([2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10, 11]) for _ in range(2)]
    dealer_cards = [random.choice([2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10, 11]) for _ in range(2)]
    game_over = False

    while not game_over:
        user_score = sum(user_cards)
        dealer_score = sum(dealer_cards)
        await ctx.send(f"당신의 카드: {user_cards}, 점수: {user_score}")
        await ctx.send(f"딜러의 첫 번째 카드: {dealer_cards[0]}")

        if user_score == 21 or dealer_score == 21 or user_score > 21:
            game_over = True
        else:
            await ctx.send("카드를 더 받으시겠습니까? (y/n)")
            response = await bot.wait_for('message', check=lambda message: message.author == ctx.author)
            if response.content.lower() == 'y':
                user_cards.append(random.choice([2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10, 11]))
            else:
                game_over = True

    while dealer_score < 17:
        dealer_cards.append(random.choice([2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10, 11]))
        dealer_score = sum(dealer_cards)

    await ctx.send(f"당신의 최종 손: {user_cards}, 점수: {user_score}")
    await ctx.send(f"딜러의 최종 손: {dealer_cards}, 점수: {dealer_score}")

    # 게임 결과 처리
    if user_score > 21:
        await ctx.send("당신이 졌습니다")
        update_user_score(ctx.author, -bet_amount)
    elif dealer_score > 21 or user_score > dealer_score:
        await ctx.send("당신이 이겼습니다")
        update_user_score(ctx.author, bet_amount)
    elif user_score < dealer_score:
        await ctx.send("당신이 졌습니다")
        update_user_score(ctx.author, -bet_amount)
    else:
        await ctx.send("무승부입니다")


# 베팅 함수
async def get_bet(ctx, user_score):
    await ctx.send(f"현재 점수: {user_score}. 베팅할 금액을 입력하세요:")
    response = await bot.wait_for('message', check=lambda message: message.author == ctx.author)

    try:
        bet_amount = int(response.content)
        if 0 < bet_amount <= user_score:
            return bet_amount
        else:
            await ctx.send("유효하지 않은 베팅입니다.")
            return None
    except ValueError:
        await ctx.send("숫자를 입력해주세요.")
        return None

# 2. 바카라, 3. 토토, 4. 슬롯 머신, 5. 파워볼 같은 방식으로 구현
# 해당 함수들도 유사한 방식으로 점수와 베팅 시스템을 적용해야 합니다.
# 이 부분은 생략하고, 블랙잭 로직을 각 게임에 맞게 변형할 수 있습니다.

# 메인 메뉴 명령어
@bot.command(name='게임')
async def game_menu(ctx):
    await ctx.send("=== 게임 메뉴 ===")
    menu_message = "\n".join([f"{i + 1}: {game}" for i, game in enumerate(games.keys())])
    await ctx.send(menu_message)
    await ctx.send("원하는 게임의 번호를 입력하세요:")

    response = await bot.wait_for('message', check=lambda message: message.author == ctx.author)
    game_choice = response.content

    if game_choice.isdigit() and 1 <= int(game_choice) <= len(games):
        selected_game = list(games.values())[int(game_choice) - 1]
        if selected_game == "blackjack":
            await blackjack(ctx)
        elif selected_game == "baccarat":
            await baccarat(ctx)
        elif selected_game == "toto":
            await toto(ctx)
        elif selected_game == "slot_machine":
            await slot_machine(ctx)
        elif selected_game == "powerball":
            await powerball(ctx)
    else:
        await ctx.send("잘못된 입력입니다. 다시 시도해주세요.")


# 봇 실행
TOKEN = MTI5OTg3MDIyMTQzNTE0MjIwNA.Gwyy59.k-AFps8q8d0vNVmtTfJ7fuMhfFzWKmNG2CbYMg  # 여기에 자신의 Discord 봇 토큰을 입력하세요.
bot.run(TOKEN)
