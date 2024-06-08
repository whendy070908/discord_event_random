import disnake
import os
import asyncio
import random
from disnake.ext import commands
admin = [] #님 디코 아이디 넣으셈
token = "" #님 봇 토큰 넣으셈
 
bot = commands.Bot(command_prefix="/", intents=disnake.Intents.all())

os.system('cls')

DEFAULT_EMOJI = "🎉"
event_participants = {}

def embeds(title: str, description: str) -> disnake.Embed:
    embed = disnake.Embed(title=title, description=description, color=0xffff)
    return embed

@bot.event
async def on_ready():
    print(f"봇 로그인 완료!\n봇 아이디: {bot.user.id} / 봇 닉네임: {bot.user}")

@bot.slash_command(name="이벤트생성", description="추첨 이벤트를 생성합니다.")
async def 이벤생성(interaction: disnake.ApplicationCommandInteraction):
    if interaction.author.id in admin or interaction.author.guild_permissions.administrator:
        modal = disnake.ui.Modal(
            title="이벤트 생성",
            custom_id="이벤_생성",
            components=[
                disnake.ui.TextInput(
                    label="이벤트 제목",
                    placeholder="이벤트의 제목을 적어주세요",
                    custom_id="이벤제목",
                    min_length=1,
                    max_length=50,
                ),
                disnake.ui.TextInput(
                    label="이벤트 설명",
                    placeholder="이벤트의 설명을 적어주세요",
                    custom_id="이벤설명",
                    min_length=1,
                    max_length=100,
                ),
                disnake.ui.TextInput(
                    label="당첨 인원수 / 참여 제한 인원",
                    placeholder="당첨 인원수와 참여 제한 인원수를 '/'로 구분하여 입력해주세요 (예: 3/200)",
                    custom_id="당첨인원제한",
                    min_length=3,
                    max_length=11,
                    style=disnake.TextInputStyle.short
                ),
                disnake.ui.TextInput(
                    label="당첨 상품",
                    placeholder="당첨 상품을 적어주세요.",
                    custom_id="당첨상품",
                    min_length=1,
                    max_length=100,
                ),
                disnake.ui.TextInput(
                    label="이모지",
                    placeholder="버튼에 넣을 이모지를 넣어주세요 (1개)",
                    custom_id="이벤모지",
                    max_length=50,
                )
            ]
        )
        await interaction.response.send_modal(modal=modal)
    else:
        await interaction.response.send_message(embed=embeds("권한 오류", "명령어를 사용할 권한이 없으십니다."), ephemeral=True)

@bot.listen('on_modal_submit')
async def handle_modal_submit(interaction: disnake.ModalInteraction):
    if interaction.custom_id == "이벤_생성":
        title = interaction.text_values["이벤제목"]
        description = interaction.text_values["이벤설명"]
        winners_and_participants = interaction.text_values["당첨인원제한"]
        prize = interaction.text_values["당첨상품"]
        emoji = interaction.text_values["이벤모지"]

        try:
            winners_count, max_participants = map(int, winners_and_participants.split('/'))
            if emoji == "":
                emoji = DEFAULT_EMOJI
            
            event_embed = disnake.Embed(title=title, description=f"{description}\n\n버튼 클릭 {max_participants}명이 넘어가면 자동 추첨됩니다.", color=0xffff)
            event_embed.set_footer(text=f"당첨상품: {prize} / 인원: 0 / {max_participants}")

            buttons = [
                disnake.ui.Button(label=f"{emoji} 참여하기", style=disnake.ButtonStyle.primary, custom_id=f"join_event_{interaction.id}")
            ]

            action_row = disnake.ui.ActionRow(*buttons)

            await interaction.response.defer()
            message = await interaction.followup.send(embed=event_embed, components=[action_row])

            event_participants[interaction.id] = {
                "max_participants": max_participants,
                "current_participants": 0,
                "winners_count": winners_count,
                "prize": prize,
                "participants": [],
                "message_id": message.id,
                "emoji": emoji
            }

        except ValueError:
            await interaction.response.send_message(embed=embeds("입력 오류", "당첨 인원수와 참여 제한 인원수는 '/'로 구분하여 정수로 입력해야 합니다."), ephemeral=True)

@bot.listen('on_button_click')
async def handle_button_click(interaction: disnake.MessageInteraction):
    event_id = int(interaction.component.custom_id.split('_')[-1])
    
    if event_id in event_participants:
        event = event_participants[event_id]
        user_id = interaction.user.id

        if event["current_participants"] >= event["max_participants"]:
            await interaction.response.send_message(embed=embeds("참여 불가", "참여 인원이 이미 다 찼습니다."), ephemeral=True)
            return

        if user_id not in event["participants"]:
            event["participants"].append(user_id)
            event["current_participants"] += 1

            await interaction.response.send_message(embed=embeds("참여 완료", f"이벤트에 성공적으로 참여하였습니다! (현재 참여 인원: {event['current_participants']}명 / {event['max_participants']}명)"), ephemeral=True)

            event_embed = interaction.message.embeds[0]
            event_embed.set_footer(text=f"당첨상품: {event['prize']} / 인원: {event['current_participants']} / {event['max_participants']}")
            await interaction.message.edit(embed=event_embed)

            if event["current_participants"] >= event["max_participants"]:
                announcement_message = await interaction.channel.send("@everyone")
                countdown_embed = await interaction.channel.send(embed=embeds("추첨 진행", f"{event['max_participants']}명 중 {event['current_participants']}명이 모였으므로 추첨을 진행하겠습니다. 3초 뒤 공개됩니다."))
                await asyncio.sleep(3)
                winners = random.sample(event["participants"], event["winners_count"])
                winner_mentions = ', '.join(f'<@{winner}>' for winner in winners)
                
                final_embed = disnake.Embed(title=event_embed.title, description=f"축하합니다! 다음 분들이 당첨되었습니다: {winner_mentions}", color=0xffff)
                final_embed.set_footer(text=f"당첨상품: {event['prize']} / 인원: {event['max_participants']} / {event['max_participants']}")
                
                buttons = [
                    disnake.ui.Button(label=f"{event['emoji']} 참여하기", style=disnake.ButtonStyle.primary, custom_id=f"join_event_{interaction.id}", disabled=True)
                ]
                action_row = disnake.ui.ActionRow(*buttons)

                await interaction.message.edit(embed=final_embed, components=[action_row])
                await announcement_message.delete()
                await countdown_embed.delete()
                await interaction.channel.send(f"{winner_mentions}")

                del event_participants[event_id]
        else:
            await interaction.response.send_message(embed=embeds("참여 실패", "이미 참여한 이벤트입니다."), ephemeral=True)

bot.run(token)
