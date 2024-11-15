import random

def deal_card():
    """카드를 랜덤으로 뽑음"""
    cards = [2, 3, 4, 5, 6, 7, 8, 9, 10, 10, 10, 10, 11]  # 10은 10, J, Q, K에 해당
    return random.choice(cards)

def calculate_hand(hand):
    """카드 값 계산 (11은 Ace, 10은 그림 카드)"""
    total = sum(hand)
    for card in hand:
        if total > 21 and card == 11:
            total -= 10  # Ace를 1로 계산
    return total

def devil_blackjack_mode(bet_amount):
    """악마의 블랙잭 모드 (룰렛 결과로 배율 결정)"""
    payout_multiplier = random.choices([1, 10, 100, 1000, 10000, 100000, 1000000], weights=[90, 5, 3, 1, 0.5, 0.25, 0.1], k=1)[0]
    print(f"룰렛 결과: 배율 x{payout_multiplier}")
    
    # 게임 진행
    player_hand = [deal_card(), deal_card()]
    dealer_hand = [deal_card(), deal_card()]
    
    print(f"플레이어 카드: {player_hand} 합: {calculate_hand(player_hand)}")
    print(f"딜러 카드: {dealer_hand} 합: {calculate_hand(dealer_hand)}")

    # 플레이어 승리 조건 (단순화)
    if calculate_hand(player_hand) > calculate_hand(dealer_hand) and calculate_hand(player_hand) <= 21:
        print(f"플레이어 승리! {bet_amount} 원에 {payout_multiplier}배")
        return bet_amount * payout_multiplier
    else:
        print(f"플레이어 패배")
        return 0

# 예시 실행
devil_blackjack_mode(100)
def extreme_lightning_baccarat(bet_amount):
    """익스트림 라이트닝 바카라 (라이트닝 카드 배율 적용)"""
    lightning_card_multiplier = random.choices([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], k=5)  # 최대 10배의 배율
    
    # 베팅 진행
    player_hand = [random.randint(1, 10), random.randint(1, 10)]
    banker_hand = [random.randint(1, 10), random.randint(1, 10)]
    
    player_total = sum(player_hand) % 10
    banker_total = sum(banker_hand) % 10
    
    print(f"플레이어 핸드: {player_hand} 합: {player_total}")
    print(f"뱅커 핸드: {banker_hand} 합: {banker_total}")
    
    if player_total > banker_total:
        multiplier = random.choice(lightning_card_multiplier)
        print(f"플레이어 승리! 배율: x{multiplier}")
        return bet_amount * multiplier
    else:
        print("뱅커 승리!")
        return 0

# 예시 실행
extreme_lightning_baccarat(200)
def extreme_lightning_storm_roulette(bet_amount):
    """익스트림 라이트닝 스톰 룰렛 (스톰 효과로 배율 결정)"""
    storm_multiplier = random.choices([1, 2, 5, 10, 20, 50, 100, 1000], weights=[80, 10, 5, 2, 1, 1, 0.5, 0.5], k=1)[0]
    print(f"라이트닝 스톰! 배율: x{storm_multiplier}")
    
    wheel = list(range(0, 37))  # 0~36 (유럽식 룰렛)
    bet_number = random.choice(wheel)
    
    print(f"룰렛 결과: {bet_number}")
    
    # 플레이어가 맞췄을 때
    if bet_number == 17:  # 예시: 17번에 베팅했다고 가정
        print(f"플레이어 승리! 배율: x{storm_multiplier}")
        return bet_amount * storm_multiplier
    else:
        print(f"플레이어 패배!")
        return 0

# 예시 실행
extreme_lightning_storm_roulette(300)
