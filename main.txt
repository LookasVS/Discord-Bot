import discord
import os
import requests
import json
import random
from replit import db
from keep_alive import keep_alive

client = discord.Client()

secretToken = os.environ['token']

test_words = [
  "gay",
  "bruh"
]

starter_replies = [
  "Cine zice, ala e..",
  "Da frate!",
  "Nane."
]

if 'responding' not in db.keys():
  db['responding'] = True

def get_quote():
  response = requests.get('https://zenquotes.io/api/random')
  json_data = json.loads(response.text)
  quote = json_data[0]['q'] + '\n - ' + json_data[0]['a']
  return(quote)

def update_enc(enc_message):
  if 'enc' in db.keys():
    enc = db['enc']
    enc.append(enc_message)
    db['enc'] = enc
  else:
    db['enc'] = [enc_message]

def delete_enc(index):
  enc = db['enc']
  if len(enc) > index:
    del enc[index]
    db['enc'] = enc

@client.event
async def on_ready():
  print('Logged in as: {0.user}'.format(client))

@client.event
async def on_message(message):
  if message.author == client.user:
    return

  msg = message.content

  if msg.startswith('%hello'):
    await message.channel.send('Salut!')

  if msg.startswith('%test'):
    quote = get_quote()
    await message.channel.send(quote)

  if db['responding']:
    options = starter_replies
    if 'enc' in db.keys():
      options = options + db['enc'].value

    if any(word in msg for word in test_words):
      await message.channel.send(random.choice(options))

  if msg.startswith('%new'):
    enc_message = msg.split('%new ', 1)[1]
    update_enc(enc_message)
    await message.channel.send('Message added!')

  if msg.startswith('%del'):
    enc = []
    if 'enc' in db.keys():
      index = int(msg.split('%del ', 1)[1])
      delete_enc(index)
      enc = db['enc']
    await message.channel.send(enc)
  
  if msg.startswith('%list'):
    enc = []
    if 'enc' in db.keys():
      enc = db['enc']
    await message.channel.send(enc)

  if msg.startswith('%responding'):
    value = msg.split('%responding ', 1)[1]
    if value.lower() == 'true':
      db['responding'] = True
      await message.channel.send("Responding is ON.")
    elif value.lower() == 'false':
      db['responding'] = False
      await message.channel.send("Responding is OFF.")

keep_alive();
client.run(secretToken)