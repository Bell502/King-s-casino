#game.py
...
def coin():
    coin_face = random.randrange(0,2)
    if coin_face == 0:
        return "홀"
    elif coin_face == 1:
        return "짝"
        #main.py
...
@bot.command()
async def 홀짝(ctx, face, money):
    userExistance, userRow = checkUser(ctx.author.name, ctx.author.id)
    forecast = coin()
    result = ""
    betting = 0
    _color = 0x000000
    if userExistance:
        cur_money = getMoney(ctx.author.name, userRow)
        if int(money) >= 10:
            if cur_money >= int(money):
                if face == "홀" or face == "짝":
                    if forecast == face:
                        result = "성공"
                        _color = 0x00ff56

                        betting = int(money)

                        modifyMoney(ctx.author.name, userRow, 0.5*betting)
                    else:
                        result = "실패"
                        _color = 0xFF0000

                        betting = int(money)
                        
                        modifyMoney(ctx.author.name, userRow, -int(betting))
                        addLoss(ctx.author.name, userRow, int(betting))

                    embed = discord.Embed(title = "홀짝게임 결과", description = result, color = _color)
                    embed.add_field(name = "배팅금액", value = betting, inline = False)
                    embed.add_field(name = "현재 자산", value = getMoney(ctx.author.name, userRow), inline = False)

                    await ctx.send(embed=embed)

                else:
                    await ctx.send("홀 또는 짝을 입력하세요")
            else:
                await ctx.send("돈이 부족합니다. 현재자산: " + str(cur_money))
        else:
            await ctx.send("10원 이상만 배팅 가능합니다.")
    else:
        await ctx.send("홀짝게임은 회원가입 후 이용 가능합니다.")
...
#user.py
...
c_loss = 5
...
def addLoss(_target, _row, _amount):
    loadFile()
    
    ws.cell(_row, c_loss).value += _amount

    saveFile()
...
def Signup(_name, _id):
    loadFile()

    _row = checkFirstRow()

    ws.cell(row=_row, column=c_name, value=_name)
    ws.cell(row=_row, column=c_id, value =hex(_id))
    ws.cell(row=_row, column=c_money, value = default_money)
    ws.cell(row=_row, column=c_lvl, value = 1)
    ws.cell(row=_row, column=c_loss, value = 0)

    saveFile()
...

def userInfo(_row):
    loadFile()

    _lvl = ws.cell(_row,c_lvl).value
    _money = ws.cell(_row,c_money).value
    _loss = ws.cell(_row,c_loss).value

    saveFile()

    return _lvl, _money, _loss
    티스토리
 스토리 서비스 목록
작업일지검색하기 
블로그 홈
프로젝트/디스코드 봇
디스코드 봇 만들기#9 - 홀짝 게임
leesihyun2037
2021. 3. 12. 17:00



블로그에는 안올라갔지만 여러가지 로직개선이 많이 있었다

아래 깃허브 링크를 참조

github.com/richardlee-kr/SuperBot

 
richardlee-kr/SuperBot

github.com
 

이제 간단한 도박 게임을 만들어보자

커맨드는 !홀짝 [예상] [거는 돈]

 

먼저 홀짝을 정하는 함수를 만든다

기존에 있던 dice.py를 game.py로 바꾸고 아래 코드를 추가했다

#game.py
...
def coin():
    coin_face = random.randrange(0,2)
    if coin_face == 0:
        return "홀"
    elif coin_face == 1:
        return "짝"
 

그리고 문제의 main.py 부분

#main.py
...
@bot.command()
async def 홀짝(ctx, face, money):
    userExistance, userRow = checkUser(ctx.author.name, ctx.author.id)
    forecast = coin()
    result = ""
    betting = 0
    _color = 0x000000
    if userExistance:
        cur_money = getMoney(ctx.author.name, userRow)
        if int(money) >= 10:
            if cur_money >= int(money):
                if face == "홀" or face == "짝":
                    if forecast == face:
                        result = "성공"
                        _color = 0x00ff56

                        betting = int(money)

                        modifyMoney(ctx.author.name, userRow, 0.5*betting)
                    else:
                        result = "실패"
                        _color = 0xFF0000

                        betting = int(money)
                        
                        modifyMoney(ctx.author.name, userRow, -int(betting))
                        addLoss(ctx.author.name, userRow, int(betting))

                    embed = discord.Embed(title = "홀짝게임 결과", description = result, color = _color)
                    embed.add_field(name = "배팅금액", value = betting, inline = False)
                    embed.add_field(name = "현재 자산", value = getMoney(ctx.author.name, userRow), inline = False)

                    await ctx.send(embed=embed)

                else:
                    await ctx.send("홀 또는 짝을 입력하세요")
            else:
                await ctx.send("돈이 부족합니다. 현재자산: " + str(cur_money))
        else:
            await ctx.send("10원 이상만 배팅 가능합니다.")
    else:
        await ctx.send("홀짝게임은 회원가입 후 이용 가능합니다.")
...
아무생각없이 짜다보니 if-else만 5개가 들어가 버렸다

로직 자체는 어렵지 않은데 조건마다 보내야하는 메세지가 다 다르다보니 이 사단이 난 것 같다

 

유저가 존재하는지 확인하고

10원 이상 배팅해야하고

현재 가지고 있는 돈이 배팅 금액보다 커야하고

홀 또는 짝을 입력하고

승패를 따지고...

 

그래도 결과적으로는 제대로 작동한다.





 

마지막으로 개인적으로 만들고 싶었던 기능중 하나인 도박으로 날린 돈 추가


 

userDB.xlsx에서 loss를 5번째 column에 추가


 

user.py의 Signup함수와 userInfo 함수를 수정 및 addLoss 함수 추가

#user.py
...
c_loss = 5
...
def addLoss(_target, _row, _amount):
    loadFile()
    
    ws.cell(_row, c_loss).value += _amount

    saveFile()
...
def Signup(_name, _id):
    loadFile()

    _row = checkFirstRow()

    ws.cell(row=_row, column=c_name, value=_name)
    ws.cell(row=_row, column=c_id, value =hex(_id))
    ws.cell(row=_row, column=c_money, value = default_money)
    ws.cell(row=_row, column=c_lvl, value = 1)
    ws.cell(row=_row, column=c_loss, value = 0)

    saveFile()
...

def userInfo(_row):
    loadFile()

    _lvl = ws.cell(_row,c_lvl).value
    _money = ws.cell(_row,c_money).value
    _loss = ws.cell(_row,c_loss).value

    saveFile()

    return _lvl, _money, _loss
 

main.py의 정보와 내정보 함수에 loss를 보내도록 수정

#main.py
...
@bot.command()
async def 내정보(ctx):
    userExistance, userRow = checkUser(ctx.author.name, ctx.author.id)

    if not userExistance:
        await ctx.send("회원가입 후 자신의 정보를 확인할 수 있습니다.")
    else:
        level, money, loss = userInfo(userRow)
        embed = discord.Embed(title="유저 정보", description = ctx.author.name, color = 0x62D0F6)
        embed.add_field(name = "레벨", value = level)
        embed.add_field(name = "보유 자산", value = money)
        embed.add_field(name = "도박으로 날린 돈", value = loss, inline = False)

        await ctx.send(embed=embed)

@bot.command()
async def 정보(ctx, user: discord.User):
    userExistance, userRow = checkUser(user.name, user.id)

    if not userExistance:
        await ctx.send(user.name  + " 은(는) 등록되지 않은 사용자입니다.")
    else:
        level, money, loss = userInfo(userRow)
        embed = discord.Embed(title="유저 정보", description = user.name, color = 0x62D0F6)
        embed.add_field(name = "레벨", value = level)
        embed.add_field(name = "보유 자산", value = money)
        embed.add_field(name = "도박으로 날린 돈", value = loss, inline = False)

        await ctx.send(embed=embed)
