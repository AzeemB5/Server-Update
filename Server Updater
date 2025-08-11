import discord
from discord.ext import commands
import json
import os
from keep_alive import keep_alive  # ✅ Keeps bot alive on Replit

# ✅ Set up Discord intents
intents = discord.Intents.default()
intents.guilds = True
intents.guild_messages = True
intents.message_content = True
intents.members = True
intents.messages = True
intents.emojis = True

# ✅ Initialize bot
bot = commands.Bot(command_prefix="!", intents=intents)

# ✅ Replace these with your actual server IDs
UPDATE_SERVER_ID = 1377392870843224144  # Update server
MAIN_SERVER_ID = 1136478773085208616    # Main server

@bot.event
async def on_ready():
    print(f"✅ Bot is online as {bot.user}")

@bot.command()
async def save(ctx):
    if ctx.guild.id != UPDATE_SERVER_ID:
        return await ctx.send("❌ This command can only be run in the update server.")

    guild = ctx.guild
    data = {
        "channels": {},
        "roles": {},
        "categories": {},
        "emojis": {}
    }

    for category in guild.categories:
        data["categories"][str(category.id)] = {"name": category.name}

    for channel in guild.channels:
        channel_data = {
            "name": channel.name,
            "type": str(channel.type),
            "category": str(channel.category.id) if channel.category else None,
            "position": channel.position
        }
        if isinstance(channel, discord.TextChannel):
            channel_data["topic"] = channel.topic or ""
        data["channels"][str(channel.id)] = channel_data

    for role in guild.roles:
        if not role.is_default():
            data["roles"][str(role.id)] = {
                "name": role.name,
                "permissions": role.permissions.value,
                "color": role.color.value
            }

    for emoji in guild.emojis:
        data["emojis"][str(emoji.id)] = {"name": emoji.name}

    with open("sync_snapshot.json", "w") as f:
        json.dump(data, f, indent=4)

    await ctx.send("✅ Snapshot saved.")

@bot.command()
async def load(ctx):
    if ctx.guild.id != MAIN_SERVER_ID:
        return await ctx.send("❌ This command can only be run in the main server.")

    try:
        with open("sync_snapshot.json", "r") as f:
            saved_data = json.load(f)
    except FileNotFoundError:
        return await ctx.send("⚠️ No snapshot found. Run `!save` in the update server first.")

    guild = ctx.guild
    change_log = []

    for category in guild.categories:
        for saved_info in saved_data["categories"].values():
            if category.name != saved_info["name"]:
                await category.edit(name=saved_info["name"])
                change_log.append(f"- Renamed category `{category.name}` to `{saved_info['name']}`")

    for channel in guild.channels:
        for saved_info in saved_data["channels"].values():
            if str(channel.type) == saved_info["type"]:
                if channel.name != saved_info["name"]:
                    await channel.edit(name=saved_info["name"])
                    change_log.append(f"- Renamed channel `{channel.name}` to `{saved_info['name']}`")
                if isinstance(channel, discord.TextChannel):
                    if channel.topic != saved_info.get("topic", ""):
                        await channel.edit(topic=saved_info["topic"])
                        change_log.append(f"- Updated topic for `{channel.name}`")
                if channel.position != saved_info.get("position", channel.position):
                    await channel.edit(position=saved_info["position"])
                    change_log.append(f"- Moved `{channel.name}` to position {saved_info['position']}")

    for role in guild.roles:
        for saved_info in saved_data["roles"].values():
            if role.name != saved_info["name"]:
                await role.edit(name=saved_info["name"])
                change_log.append(f"- Renamed role `{role.name}` to `{saved_info['name']}`")
            if role.permissions.value != saved_info["permissions"]:
                perms = discord.Permissions(saved_info["permissions"])
                await role.edit(permissions=perms)
                change_log.append(f"- Updated permissions for role `{saved_info['name']}`")
            if role.color.value != saved_info["color"]:
                await role.edit(color=discord.Color(saved_info["color"]))
                change_log.append(f"- Updated color for role `{saved_info['name']}`")

    for emoji in guild.emojis:
        for saved_info in saved_data["emojis"].values():
            if emoji.name != saved_info["name"]:
                change_log.append(f"- Emoji `{emoji}` should be renamed to `{saved_info['name']}` (manual step)")

    if change_log:
        await ctx.send("🔄 Changes applied:\n" + "\n".join(change_log))
    else:
        await ctx.send("✅ No changes detected.")

# ✅ Keep bot alive (Replit only)
keep_alive()

# ✅ Run the bot using environment variable
bot.run(os.getenv("DISCORD_BOT_TOKEN"))
