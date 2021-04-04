# BiscuitBot
import discord
from discord.ext import commands
import random
import os
import json
import requests
import aiofiles
from keep_alive import keep_alive

os.getcwd()
#os.chdir('')
client = commands.Bot(command_prefix = "b!")
#client.remove_command('help')
client.warnings = {}
filters = ["Fuck","fuck","bitch","Bitch","Asshole","Mother fucker","asshole"]
snipe_message_author = {}
snipe_message_content = {}

@client.event
async def on_ready():
  for guild in client.guilds:
    client.warnings[guild.id] = {}
    async with aiofiles.open(f"{guild.id}.txt", mode="a") as temp:
      pass

    async with aiofiles.open(f"{guild.id}.txt", mode="r") as file:
      lines = await file.readlines()

      for line in lines:
        data = line.split(" ")
        member_id = int(data[0])
        admin_id = int(data[1])
        reason = " ".join(data[2:]).strip("\n")

        try:
          client.warnings[guild.id][member_id][0] += 1
          client.warnings[guild.id][member_id][1].append((admin_id, reason))

        except KeyError:
          client.warnings[guild.id][member_id] = [1, [(admin_id, reason)]]

  print('Bot is online')
  await client.change_presence(activity=discord.Game(name="Eating Biscuits :) | b!help"))

@client.event
async def on_guild_join(guild):
  client.warnings[guild.id] = {}

@client.command()
@commands.has_permissions(administrator=True)
async def warn(ctx, member: discord.Member=None, *, reason=None):
  if member is None:
    return await ctx.send("The provided member could not be found or you didn't to provide one.")

  if reason is None:
    return await ctx.send("Please provide a reason for warning this user.")

    try:
      first_warning = False
      client.warnings[ctx.guild.id][member.id][0] += 1
      client.warnings[ctx.guild.id][member.id][1].append((ctx.author.id, reason))

    except KeyError:
      first_warning = True
      client.warnings[ctx.guild.id][member.id] = [1, [(ctx.author.id, reason)]]

    count = client.warnings[ctx.guild.id][member.id][0]

    async with aiofiles.open(f"{member.guild.id}.txt", mode="a") as file:
      await file.write(f"{member.id} {ctx.author.id} {reason}\n")

      await ctx.send(f'{member.mention} has been warned successfully, Reason: {reason}. They now have {count} warnings.')
      await member.send(f'You have been warned in {ctx.guild.name} by {ctx.author.name}, Reason: {reason}. You now have {count} warnings ')
      #await ctx.send(f"{member.mention} has {count} {'warning' if first_warning else 'warnings'}.")

@client.command()
@commands.has_permissions(administrator=True)
async def checkwarn(ctx, member: discord.Member=None):
  if member is None:
    return await ctx.send("The provided member could not be found or you didn't provide one.")

    embed = discord.Embed(title=f"Displaying Warnings for {member.name}", description="", colour=discord.Colour.red())
    try:
      i = 1
      for admin_id, reason in client.warnings[ctx.guild.id][member.id][1]:
        admin = ctx.guild.get_member(admin_id)
        embed.description += f"**Warning {i}** given by: {admin.mention} for: *'{reason}'*.\n"
        i += 1

        await ctx.send(embed=embed)

    except KeyError: # no warnings
      await ctx.send("This user has no warnings.")

@client.event
async def on_member_join(ctx, *,member):
  print(f'{member} has joined {ctx.guild.name}')
  await member.send(f'Welcome to {ctx.guild.name}!! Hope you have a great time here')

@client.event
async def on_member_remove(ctx, *,member):
  print(f'{member} has left {ctx.guild.name}')

@client.command()
async def ping(ctx):
  await ctx.send(f'pong! {round(client.latency * 1000)}ms')

@client.command()
async def pong(ctx):
  await ctx.send(f'ping! {round(client.latency * 1000)}ms')

@client.command(aliases = ['8ball'])
async def _8ball(ctx, *, question):
  responses = ['It is certain.',
              'Without a doubt.',
              'You may rely on it.',
              'Yes definitely.',
              'It is decidedly so.',
              'As I see it, yes.',
              'Most likely.',
              'Yes.',
              'Outlook good.',
              'Signs point to yes Neutral Answers.',
              'nope.',
              'no.',
              'never.',
              'maybe.',
              'probably.',
              'probably not.',
              'could be.']
  await ctx.send(f'question: {question}\nAnswer: {random.choice(responses)}')

@client.command(aliases = ['k'])
@commands.has_permissions(kick_members = True)
async def kick(ctx, member : discord.Member, *,reason='No reason provided'):
  await member.kick(reason=reason)
  await ctx.send(f'{member} has been kicked from the server successfully, Reason: ' + reason)
  await member.send(f'You have been kicked from {ctx.guild.name} by {ctx.author.name}, Reason: ' + reason)
client.command(aliases = ['ban'])
@commands.has_permissions(ban_members = True)
async def Ban(ctx, member : discord.Member, *,reason='No reason provided'):
  await  member.ban(reason=reason)
  await ctx.send(f'{member} has been banned from the server uccessfully, Reason: ' + reason)
  await member.send(f'You have been banned from {ctx.guild.name} by {ctx.author.name}, Reason: ' + reason)

@client.command()
@commands.has_permissions(ban_members = True)
async def unban(ctx, *, member: discord.Member):
  banned_users = await ctx.guild.bans()
  member_name, member_disc = member.split('#')

  for banned_entry in banned_users:
    user = banned_entry.user

    if(user.name, user.discriminator)==(member_name,member_disc):
      await ctx.guild.unban(user)
      await ctx.send(f"member has been unbanned by {ctx.author.name}")
      await member.send(f'you have been unbanned from {ctx.guild.name} by {ctx.author.name}')
      return

      await ctx.send(member+" was not found")

@client.command()
async def op(ctx, *, member: discord.Member=None):
  if member == None:
    member = ctx.author
    await ctx.send(f'I do not lie but {ctx.author.mention} is the awesome')
    return
  await ctx.send(f'I do not lie but {member.mention} is awesome :)')


@client.command()
async def message(ctx, member: discord.Member, *, message="None"):
  await member.send(f'{ctx.author.name} sent a message for you. MESSAGE: {message}')
  await ctx.send('Message has been delivered!!')

@client.command()
async def mention(ctx, member: discord.Member, *, reason='None'):
  await ctx.send(f'{ctx.author.name} has mentioned {member.mention} in {ctx.guild.name}, Reason: {reason}')
  await member.send(f'{ctx.author.name} mentioned you in {ctx.guild.name}, Reason: {reason}')

@client.command(aliases = ['m'])
@commands.has_permissions(kick_members=True)
async def mute(ctx, member: discord.Member, *, reason="No Reason Provided"):
  #await <discord.Member>.add_roles(muted_role_id)
  #muted_role = discord.utils.get(ctx.guild.roles,name="Muted")
  #await member.add_roles(muted_role)
  muted_role = discord.utils.get(ctx.guild.roles,name="Muted")
  await member.add_roles(muted_role)
  #oth_embed = discord.Embed(title="mute", description=f'muted-{member.mention} by {ctx.author.name}, Reason: {reason}')
  await ctx.send(f"Muted {member.mention} successfully!, Reason: {reason}")
  #await ctx.send(oth_embed)
  await member.send(f" You have been muted from: {ctx.guild.name} by {ctx.author.name} Reason: {reason}")

@client.command(description="Unmutes a specified user.")
@commands.has_permissions(kick_members=True)
async def unmute(ctx, member: discord.Member):
  mutedRole = discord.utils.get(ctx.guild.roles, name="Muted")

  await member.remove_roles(mutedRole)
  await ctx.send(f'Unmuted {member.mention} successfully!')
  await member.send(f'you have unmuted from {ctx.guild.name} by {ctx.author.name}')
  #embed = discord.Embed(title="unmute", description=f"unmuted-{member.mention}", colour=discord.Colour.light_gray())

@client.event
async def on_message(message):
  for filter in filters:
    if filter in message.content:
      await message.delete()

    """if message.content == "f" or "F":
        await message.channel.send("**f**")

    elif message.content == "lol":
        await message.channel.send('Hey tell me a joke! I wanna laugh aswell')"""

  await client.process_commands(message)

"""@client.event
async def on_message(message):
    #if message.author.bot:
    #    return
    for filter in filters:
        if filter in message.content:
            await message.delete()

    if message.content == "f" or "F":
        await message.channel.send("**f**")

    elif message.content == "lol":
        await message.channel.send('Hey tell me a joke! I wanna laugh aswell')

    await client.process_commands(message)
"""
@client.command()
async def addfilter(ctx,*,word):
  filters.append(word)
  await ctx.send("Successfully added the filter to this server")

@client.command()
async def removefilter(ctx,*,word):
  if word not in filters:
    await ctx.send("That filter is not present")
    return
    filters.remove(word)
    await ctx.send("Successfully removed the filter to this server")

@client.event
async def on_message_delete(message):
  snipe_message_author[message.channel.id] = message.author
  snipe_message_content[message.channel.id] = message.content
  #await sleep(60)
  del snipe_message_author[message.channel.id]
  del snipe_message_content[message.channel.id]

@client.command(name = 'snipe')
async def snipe(ctx):
  channel = ctx.channel
  try:
    em = discord.Embed(name = f"Last deleted message in #{channel.name}", description = snipe_message_content[channel.id])
    em.set_footer(text = f"This message was sent by {snipe_message_author[channel.id]}")
    await ctx.send(embed = em)
  except: #This piece of code is run if the bot doesn't find anything in the dictionary
    await ctx.send(f"There are no recently deleted messages in #{channel.name}")
    await client.process_commands(message)

"""@client.command(pass_context=True)
async def help(ctx):
  author = ctx.message.author
  embed = discord.Embed(
  colour = discord.Colour.orange()
  )
  embed.set_author(name="Help")
  embed.add_field(name='b!ping', value="returns pong!")

  await client.send_message(author, embed=embed)"""

@client.command(aliases = ['bal'])
async def balance(ctx):
  await open_account(ctx.author)
  
  users = await get_bank_data()
  em = discord.Embed()

async def open_account(user):
  users = await get_bank_data()

  if str(user.id) in users:
    return False

  else:
    users[str(user.id)]['wallet'] = 0
    users[str(user.id)]['bank'] = 0

  with open('mainbank.json','w') as f:
    json.dump(users,f)

async def get_bank_data():
  with open('mainbank.json','r') as f:
    user = json.load(f)

  return users

keep_alive()
client.run(os.getenv('Token'))


