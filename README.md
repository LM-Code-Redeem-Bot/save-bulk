```python
# -*- coding: utf-8 -*-
from flask import Flask
from threading import Thread

app = Flask('')


@app.route('/')
def home():
    return "Hello. I am alive!"


def run():
    app.run(host='0.0.0.0', port=8080)


def keep_alive():
    t = Thread(target=run)
    t.start()


import discord
import os
import requests
import time
from threading import Thread
import pandas as pd
import numpy as np
import matplotlib.pyplot as plt
import six
import asyncio

codes = []

admins = list(str(os.getenv('admin')).split(","))
its_count = 30

client = discord.Client(intents=discord.Intents.default())


def render_mpl_table(data,
                     fname,
                     col_width=3.0,
                     row_height=0.625,
                     font_size=14,
                     header_color='#000',
                     row_colors=['#f1f1f2', 'w'],
                     edge_color='w',
                     bbox=[0, 0, 1, 1],
                     header_columns=0,
                     ax=None,
                     **kwargs):
    if ax is None:
        size = (np.array(data.shape[::-1]) + np.array([0, 1])) * np.array(
            [col_width, row_height])
        fig, ax = plt.subplots(figsize=size)
        ax.axis('off')

    mpl_table = ax.table(cellText=data.values,
                         bbox=bbox,
                         colLabels=data.columns,
                         cellLoc='center',
                         **kwargs)

    mpl_table.auto_set_font_size(False)
    mpl_table.set_fontsize(font_size)

    for k, cell in six.iteritems(mpl_table._cells):
        cell.set_edgecolor(edge_color)
        if k[0] == 0 or k[1] < header_columns:
            cell.set_text_props(weight='bold', color='w')
            cell.set_facecolor(header_color)
        else:
            cell.set_facecolor(row_colors[k[0] % len(row_colors)])
    fig.savefig(fname)


def sub_redeem(i, temp,names, code, ops):
    global its_count
    its_count=its_count+1
    j = 0
    c = 0
    while (c != 1):
        try:
            res = requests.get("https://lmbot-" + str(its_count) +
                               ".vercel.app/" + str(temp[i - 1]) + "/" + str(code))
            # print(res.text, str(its_count), str(temp[i - 1]), str(code), res.status_code)
            tmp = list(str(res.text).split("*"))
            if (res.text[0] == "*"):
                j = j - 1
            elif (tmp[0] == "Enter your IGG ID"):
                j = j - 1
            elif (tmp[0] == "Enter gift code"):
                j = j - 1
            elif (res.status_code == 504):
                j = j - 1
            else:
                print(res.text, res.status_code, "server :- ", its_count)
                channel = client.get_channel(int(os.getenv('chanlnel_id')))
                send_msg(
                    channel, "\n----------------------------------------------------------\nGame Name :- " + str(names[i - 1]) + ", Code :- " +
                    str(code) + ", Result From IGG :- \n" + str(res.text[:res.text.find(str(temp[i - 1]))-1])+"\n----------------------------------------------------------\n")
                c = 1
            j = j + 1
        except Exception as e:
            print("Error :- ", e)
            channel = client.get_channel(int(os.getenv('chanlnel_id')))
            send_msg(channel, "Serious Issue Occured from IGG :-\n" + str(e))
            break
    its_count=its_count-1


def redeem(code, ops):
    codes.append(code)
    print(time.ctime())
    data = pd.read_csv(os.getenv('data_sheet'))
    temp = list(data['ID'])
    names=list(data['Game Name'])
    c = len(temp)
    for i in range(1, c + 1):
        # sub_redeem(i,temp,code):
        background_thread = Thread(target=sub_redeem,
                                   args=(
                                       i,
                                       temp,
                                      names,
                                       code,
                                       ops,
                                   ))
        background_thread.start()
        # response = requests.get("https://lmcrbot-" + str(i) +
        #                         ".vercel.app/" + str(temp[i - 1]) + "/" +
        #                         str(code))
        # print("Bot-", i, response.status_code, time.ctime())
        # if (response.status_code == 504):
        #     time.sleep(2)
        #     response = requests.get("https://lmcrbot-" + str(i) +
        #                             ".vercel.app/" + str(temp[i - 1]) +
        #                             "/" + str(code))
        #     print("Bot-", i, response.status_code, time.ctime())
    # print("🚀", code, "Redemeed Successfully...!!!")
    send_msg(
        ops, "🚀 " + str(code) +
        " Redemeed Successfully...!!!\n==================================")
    embed=discord.Embed(
    title="Code Gift Summary",
        description="Check Here For Gift You Received",
        color=discord.Color.blue())
    embed.set_author(name="Gift Bot", icon_url="data:image/png;base64,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")
    embed.set_thumbnail(url="https://cdn.dribbble.com/users/42048/screenshots/8350927/media/23289b76ac7353ad4f0d0619ce6e9f2d.gif")
    embed.add_field(name="****", value="Surround your text in asterisks (\*)", inline=False)
    embed.set_footer(text="Please Keep Bot Secret...!!!")
    gift_msg(ops,embed)


def send_msg(ops, msg):
    client.loop.create_task(ops.send(msg))

def gift_msg(ops, msg):
    client.loop.create_task(ops.send(embed=msg))


@client.event
async def on_ready():
    await client.change_presence(activity=discord.Game(
        name="With Lords Mobile Gift Codes"))
    print('We have logged in as {0.user}'.format(client))


@client.event
async def on_message(message):
    if message.author == client.user:
        return

    if message.content.startswith(
        ('&Code@history', '&Code@History', '&code@History', '&code@history',
         '&CODE@HISTORY')):
        if str(message.author.name) in admins:
            msg1 = '🧿{0.author.mention} History of Codes Till Today As Follows...'.format(
                message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')
            if (len(list(
                    pd.read_csv('code_history.csv')["Codes History"])) == 0):
                await message.channel.send("🤐    Co History Is Empty...!!!")
            else:
                await message.channel.send(
                    "\n ----------------------------------------------------------"
                )
                c = 1
                for i in list(
                        pd.read_csv('code_history.csv')["Codes History"]):
                    await message.channel.send('\n' + str(c) + ') ' + str(i))
                    c = c + 1
                await message.channel.send(
                    "\n ----------------------------------------------------------"
                )
                await message.channel.send('=================================='
                                           )
                render_mpl_table(pd.read_csv('code_history.csv'),
                                 "codes.png",
                                 header_columns=0,
                                 col_width=2.0)
                await message.channel.send(file=discord.File('codes.png'))
            await message.channel.send('==================================')
            file_path = "codes.png"
            if os.path.isfile(file_path):
              os.remove(file_path)
        else:
            msg1 = '🧐 {0.author.mention} Warning Its Only For Admins...!!!'.format(
                message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')

    if message.content.startswith(('^help', '^Help')):
        msg1 = '🚀 Hey Dude, Here are bot commands for you !!! {0.author.mention}'.format(
            message)
        await message.channel.send('==================================')
        await message.channel.send(msg1)
        await message.channel.send('==================================')
        await message.channel.send(
            '✅ Bot Commands :-\n1)  ^help\n2) ^hi\n3) ^code LM_Code')
        await message.channel.send('==================================')

    if message.content.startswith(('&help', '&Help')):
        if str(message.author.name) in admins:
            msg1 = '☯ Hey Dude, Here are bot Admin commands for you !!! {0.author.mention}'.format(
                message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')
            await message.channel.send(
                '🎉 Bot Commands :-\n1)  &help\n2) &code@history\n3) &data@history\n4) &code@reset '
            )
            await message.channel.send('==================================')
        else:
            msg1 = '🧐 {0.author.mention} Warning Its Only For Admins...!!!'.format(
                message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')

    if message.content.startswith(('^hi', '^Hi', '^Hii', '^hii')):
        msg1 = '✨ Hey Dude, Welcome Here !!! {0.author.mention}'.format(
            message)
        await message.channel.send('==================================')
        await message.channel.send(msg1)
        await message.channel.send('==================================')

    if message.content.startswith(('^code', '^Code','^^code', '^^Code')):
        l = str(message.content)
        li = l.split()
        if (len(li) == 1):
            await message.channel.send("👀 No Code Entered...!!!")
        elif str(li[1].upper()) in codes:
            await message.channel.send(
                "🙄 Duplication Of Code Is Not Allowed...!!!")
        else:
            li[1] = li[1].upper()
            old_code = pd.read_csv('code_history.csv')
            new_code = pd.DataFrame({"Codes History": [li[1]]})
            old_code = pd.concat([old_code, new_code])
            # old_code.append(new_code)
            old_code.to_csv('code_history.csv', index=False)
            await message.channel.send('==================================')
            await message.channel.send(
                '🔰 Code :- {}, Added For Redeemption'.format(li[1]))
            await message.channel.send('==================================')
            await message.channel.send(
                '⛄ Please Wait For a Minute Untill You Enter Another...!!!')
            await message.channel.send('==================================')
            # redeem(li[1])
            background_thread = Thread(target=redeem,
                                       args=(
                                           li[1],
                                           message.channel,
                                       ))
            background_thread.start()
            # await message.channel.send("🚀" + str(li[1]) +
            #                            " ,Redemeed Successfully...!!!")

    if message.content.startswith(
        ('&Data@history', '&data@history', '&Data@History', '&DATA@HISTORY',
         '&data@History')):
        if str(message.author.name) in admins:
            msg1 = '🧿{0.author.mention} Data Is As Follows...'.format(message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')
            if (len(list(pd.read_csv(os.getenv('data_sheet'))["ID"])) == 0):
                await message.channel.send('👀 Data History Is Empty...!!!')
                await message.channel.send('=================================='
                                           )
            else:
                await message.channel.send(
                    pd.read_csv(
                        os.getenv('data_sheet')).to_markdown(index=False))
                await message.channel.send('=================================='
                                           )
                render_mpl_table(pd.read_csv(os.getenv('data_sheet')),
                                 "data.png",
                                 header_columns=0,
                                 col_width=2.0)
                await message.channel.send(file=discord.File('data.png'))
                await message.channel.send('=================================='
                                           )
                file_path = "data.png"
                if os.path.isfile(file_path):
                  os.remove(file_path)
        else:
            msg1 = '🧐 {0.author.mention} Warning Its Only For Admins...!!!'.format(
                message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')

    if message.content.startswith(('&code@reset', '&Code@Reset', '&Code@reset',
                                   '&code@Reset', '&CODE@RESET')):
        if str(message.author.name) in admins:
            await message.channel.send('==================================')
            new_code = pd.DataFrame(columns=["Codes History"])
            new_code.to_csv('code_history.csv', index=False)
            await message.channel.send("Code History Deleted Successfully..!!!"
                                       )
            await message.channel.send('==================================')
        else:
            msg1 = '🧐 {0.author.mention} Warning Its Only For Admins...!!!'.format(
                message)
            await message.channel.send('==================================')
            await message.channel.send(msg1)
            await message.channel.send('==================================')
    
    if message.content.startswith(('!code', '!Code','!CODE')):
        l = str(message.content)
        li = l.split()
        if (len(li) == 1):
            await message.channel.send("👀 No Code Entered...!!!")
        elif str(li[1].upper()) in codes:
            await message.channel.send(
                "🙄 Duplication Of Code Is Not Allowed...!!!")
        else:
            li[1] = li[1].upper()
            old_code = pd.read_csv('code_history.csv')
            new_code = pd.DataFrame({"Codes History": [li[1]]})
            old_code = pd.concat([old_code, new_code])
            # old_code.append(new_code)
            old_code.to_csv('code_history.csv', index=False)
            background_thread = Thread(target=redeem,
                                       args=(
                                           li[1],
                                           message.channel,
                                       ))
            background_thread.start()
    
    
    
    
    
keep_alive()
loop = asyncio.get_event_loop()
loop.create_task(client.start(os.getenv('TOKEN')))
Thread(target=loop.run_forever).start()

```
