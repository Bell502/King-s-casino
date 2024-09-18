pip install discord.py
import discord
from discord.ext import commands
 
bot = commands.Bot(command_prefix='#')
 
@bot.event
async def on_ready():
    print(f'Login bot: {bot.user}')
 
@bot.command()
async def hello(message):
    await message.channel.send('Hi!')
 
bot.run('#OTY5MTUyNz')
@bot.command()
async def 회원가입(ctx):
    print(ctx.author.name)
    print(ctx.author.id)
    #user.py

from openpyxl import load_workbook, Workbook

c_name = 1
c_id = 2
c_money = 3
c_lvl = 4

default_money = 10000

wb = load_workbook("userDB.xlsx")
ws = wb.active

def signup(_name, _id):
    ws.cell(row=2, column=c_name, value=_name)
    ws.cell(row=2, column=c_id, value =_id)
    ws.cell(row=2, column=c_money, value = default_money)
    ws.cell(row=2, column=c_lvl, value = 1)

    wb.save("userDB.xlsx")
@bot.command()
async def 회원가입(ctx):
    signup(ctx.author.name, ctx.author.id)
    ...

def checkRow():
    for row in range(2, ws.max_row + 1):
        if ws.cell(row,1).value is None:
            return row
            break
    #return ws.max_row 불안정하지만 훨씬 간단함

def signup(_name, _id):
    _row = checkRow()
    
    ws.cell(row=_row, column=c_name, value=_name)
    ws.cell(row=_row, column=c_id, value =_id)
    ws.cell(row=_row, column=c_money, value = default_money)
    ws.cell(row=_row, column=c_lvl, value = 1)
    
...
#user.py

...
def checkName(_name, _id):
    for row in range(2, ws.max_row+1):
        if ws.cell(row,1).value == _name and ws.cell(row,2).value == _id:
            break
            return False
        else:
            return True
            break
...
#main.py

...
@bot.command()
async def 회원가입(ctx):
    #print(ctx.author.name)
    #print(ctx.author.id)
    if checkName(ctx.author.name, ctx.author.id):
        signup(ctx.author.name, ctx.author.id)
        await ctx.send("회원가입이 완료되었습니다.")
    else:
        await ctx.send("이미 가입하셨습니다.")
...
#user.py
...
def delete():
ws.delete_rows(2,ws.max_row)
wb.save("userDB.xlsx")
#main.py
...
@bot.command()
async def reset(ctx):
delete()
...
#user.py
...
def getMoney(_name, _id):
	loadFile()
    
    for row in range(2, ws.max_row+2):
    	if ws.cell(row, c_name).value == _name and ws.cell(row,c_id).value == hex(_id):
        	return ws.cell(row,c_money).value
            break
        else:
        	return 0
            break
 ...
 @bot.command()
async def 송금(ctx, user: discord.User, money):
    if checkName(user.name, user.id):
        await ctx.send("등록되지 않는 사용자입니다.")
    else:
        if getMoney(ctx.author.name, ctx.author.id) >= int(money):
            await ctx.send("송금")
        else:
            await ctx.send("돈이 충분하지 않습니다.")
#user.py
...
def remit(sender, s_id, receiver, r_id, _amount):
    loadFile()
    
    receiver_row = findRow(receiver, r_id)
    sender_row = findRow(sender, s_id)
    
    ws.cell(receiver_row, c_money).value += int(_amount)
    ws.cell(sender_row, c_money).value -= int(_amount)

    saveFile()
...
#main.py
...
@bot.command()
async def 송금(ctx, user: discord.User, money):
    if findRow(user.name, user.id) == None:
        await ctx.send("등록되지 않는 사용자입니다.")
    else:
        s_money = getMoney(ctx.author.name, ctx.author.id)
        r_money = getMoney(user.name, user.id)

        if s_money >= int(money):
            remit(ctx.author.name, ctx.author.id, user.name, user.id, money)

            embed = discord.Embed(title="송금 완료", description = "송금된 돈: " + money, color = 0x77ff00)
            embed.add_field(name = "보낸 사람: " + ctx.author.name, value = "현재 자산: " + str(getMoney(ctx.author.name, ctx.author.id)))
            embed.add_field(name = ":arrow_forward:", value = "")
            embed.add_field(name="받은 사람: " + user.name, value="현재 자산: " + str(getMoney(user.name, user.id)))
                    
            await ctx.send(embed=embed)
        else:
            await ctx.send("돈이 충분하지 않습니다.")
 ...
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

...

