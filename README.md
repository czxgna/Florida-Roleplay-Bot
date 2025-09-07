import discord
from discord.ext import commands
import datetime

intents = discord.Intents.default()
intents.message_content = True
intents.members = True

bot = commands.Bot(command_prefix="!", intents=intents)

# A simple in-memory staff list
staff_list = set()

# Sessions Tracking
active_sessions = {}
active_shifts = {}


# Helper to create timestamp string
def time_now():
    return datetime.datetime.now().strftime('%Y-%m-%d %H:%M:%S')


# Session
@bot.command()
async def start_session(ctx):
    user = ctx.author
    if user.id in active_sessions:
        await ctx.send(embed=discord.Embed(
            title="Session Already Active",
            description=f"{user.mention}, you already started a session at `{active_sessions[user.id]}`.",
            color=discord.Color.orange()
        ))
        return

    active_sessions[user.id] = time_now()
    embed = discord.Embed(
        title="Session Started",
        description=f"{user.mention} started a session.",
        color=discord.Color.green()
    )
    embed.set_footer(text=f"Time: {active_sessions[user.id]}")
    await ctx.send(embed=embed)


@bot.command()
async def end_session(ctx):
    user = ctx.author
    if user.id not in active_sessions:
        await ctx.send(embed=discord.Embed(
            title="No Active Session",
            description=f"{user.mention}, you have no active session to end.",
            color=discord.Color.red()
        ))
        return

    start_time = active_sessions.pop(user.id)
    embed = discord.Embed(
        title="Session Ended",
        description=f"{user.mention} ended their session.",
        color=discord.Color.blue()
    )
    embed.set_footer(text=f"Started at: {start_time} | Ended at: {time_now()}")
    await ctx.send(embed=embed)


# Shift
@bot.command()
async def start_shift(ctx):
    user = ctx.author
    if user.id in active_shifts:
        await ctx.send(embed=discord.Embed(
            title="Shift Already Active",
            description=f"{user.mention}, you already started a shift at `{active_shifts[user.id]}`.",
            color=discord.Color.orange()
        ))
        return

    active_shifts[user.id] = time_now()
    embed = discord.Embed(
        title="Shift Started",
        description=f"{user.mention} started a staff shift.",
        color=discord.Color.green()
    )
    embed.set_footer(text=f"Time: {active_shifts[user.id]}")
    await ctx.send(embed=embed)


@bot.command()
async def end_shift(ctx):
    user = ctx.author
    if user.id not in active_shifts:
        await ctx.send(embed=discord.Embed(
            title="No Active Shift",
            description=f"{user.mention}, you have no active shift to end.",
            color=discord.Color.red()
        ))
        return

    start_time = active_shifts.pop(user.id)
    embed = discord.Embed(
        title="Shift Ended",
        description=f"{user.mention} ended their staff shift.",
        color=discord.Color.blue()
    )
    embed.set_footer(text=f"Started at: {start_time} | Ended at: {time_now()}")
    await ctx.send(embed=embed)


# Staff Punishment
@bot.command()
@commands.has_permissions(kick_members=True)
async def warn(ctx, member: discord.Member, *, reason="No reason provided"):
    embed = discord.Embed(
        title="⚠️ Staff Warning",
        description=f"{member.mention} has been warned.",
        color=discord.Color.orange()
    )
    embed.add_field(name="Reason", value=reason, inline=False)
    embed.set_footer(text=f"Issued by: {ctx.author}")
    await ctx.send(embed=embed)


@bot.command()
@commands.has_permissions(kick_members=True)
async def kick(ctx, member: discord.Member, *, reason="No reason provided"):
    await member.kick(reason=reason)
    embed = discord.Embed(
        title="👢 Member Kicked",
        description=f"{member.mention} was kicked from the server.",
        color=discord.Color.red()
    )
    embed.add_field(name="Reason", value=reason, inline=False)
    embed.set_footer(text=f"Issued by: {ctx.author}")
    await ctx.send(embed=embed)


@bot.command()
@commands.has_permissions(ban_members=True)
async def ban(ctx, member: discord.Member, *, reason="No reason provided"):
    await member.ban(reason=reason)
    embed = discord.Embed(
        title="⛔ Member Banned",
        description=f"{member.mention} was banned from the server.",
        color=discord.Color.dark_red()
    )
    embed.add_field(name="Reason", value=reason, inline=False)
    embed.set_footer(text=f"Issued by: {ctx.author}")
    await ctx.send(embed=embed)


# STAFF MANAGEMENT COMMAND
@bot.command()
@commands.has_permissions(manage_roles=True)
async def add_staff(ctx, member: discord.Member):
    staff_list.add(member.id)
    embed = discord.Embed(
        title=" Staff Added",
        description=f"{member.mention} has been added to the staff list.",
        color=discord.Color.green()
    )
    await ctx.send(embed=embed)


@bot.command()
@commands.has_permissions(manage_roles=True)
async def remove_staff(ctx, member: discord.Member):
    staff_list.discard(member.id)
    embed = discord.Embed(
        title=" Staff Removed",
        description=f"{member.mention} has been removed from the staff list.",
        color=discord.Color.red()
    )
    await ctx.send(embed=embed)


@bot.command()
async def list_staff(ctx):
    if not staff_list:
        await ctx.send("No staff members currently registered.")
        return

    members = []
    for member_id in staff_list:
        member = ctx.guild.get_member(member_id)
        if member:
            members.append(member.mention)

    embed = discord.Embed(
        title="📋 Current Staff",
        description="\n".join(members) if members else "No valid members found.",
        color=discord.Color.blue()
    )
    await ctx.send(embed=embed)


# Debugging
@bot.event
async def on_command_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send(embed=discord.Embed(
            title="❌ Permission Denied",
            description="You don't have permission to use this command.",
            color=discord.Color.red()
        ))
    elif isinstance(error, commands.MissingRequiredArgument):
        await ctx.send(embed=discord.Embed(
            title="⚠️ Missing Argument",
            description=str(error),
            color=discord.Color.orange()
        ))
    elif isinstance(error, commands.BadArgument):
        await ctx.send(embed=discord.Embed(
            title="⚠️ Invalid Argument",
            description=str(error),
            color=discord.Color.orange()
        ))
    else:
        raise error  # Re-raise unhandled errors


# bot token
bot.run("")
