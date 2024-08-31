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
await ctx.send("돈이 부족합니다. 현재자산: " + str(cur_money))await ctx.send("돈이 부족합니다. 현재자산: " + str(cur_money))
else:
   await ctx.send("10원 이상만 배팅 가능합니다.")
   else:
   await ctx.send("홀짝게임은 회원가입 후 이용 가능합니다.")
   
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
