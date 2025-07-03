# IRC Server Evaluation Guide

## Connection & Authentication
1. **PASS** - Password Authentication
   - Usage: `PASS <password>`
   - Test: Client must provide correct password to authenticate
   - Error cases:
     * ERR_NEEDMOREPARAMS if no password provided
     * ERR_ALREADYREGISTERED if already authenticated
     * ERR_PASSWDMISMATCH if password is incorrect

2. **NICK** - Set/Change Nickname
   - Usage: `NICK <nickname>`
   - Rules: Nicknames must be:
     * 1-9 characters long
     * Start with a letter
     * Contain only letters, numbers, '-' or '_'
   - Error cases:
     * ERR_NONICKNAMEGIVEN if no nickname provided
     * ERR_ERRONEUSNICKNAME if invalid format
     * ERR_NICKNAMEINUSE if already taken

3. **USER** - Set Username
   - Usage: `USER <username> 0 * :<realname>`
   - Test: Required for complete registration
   - Error cases:
     * ERR_NEEDMOREPARAMS if insufficient parameters
     * ERR_ALREADYREGISTERED if already registered

## Channel Operations
1. **JOIN** - Join a Channel
   - Usage: `JOIN <channel> [key]`
   - Test cases:
     * Creating new channel (first user becomes operator)
     * Joining existing channel
     * Joining with key (if +k mode set)
   - Error cases:
     * ERR_NEEDMOREPARAMS if no channel specified
     * ERR_BANNEDFROMCHAN if banned
     * ERR_INVITEONLYCHAN if +i mode set
     * ERR_BADCHANNELKEY if wrong key
     * ERR_CHANNELISFULL if +l limit reached

2. **PART** - Leave a Channel
   - Usage: `PART <channel> [reason]`
   - Test: 
     * Leaving with message
     * Channel deletion when empty
   - Error cases:
     * ERR_NOTONCHANNEL if not in channel
     * ERR_NOSUCHCHANNEL if channel doesn't exist

3. **KICK** - Remove User from Channel
   - Usage: `KICK <channel> <nickname> [reason]`
   - Test:
     * Only operators can kick
     * Kick message broadcast
   - Error cases:
     * ERR_CHANOPRIVSNEEDED if not operator
     * ERR_NOTONCHANNEL if target not in channel

4. **INVITE** - Invite User to Channel
   - Usage: `INVITE <nickname> <channel>`
   - Test:
     * Only operators can invite
     * Invited users can join +i channels
   - Error cases:
     * ERR_CHANOPRIVSNEEDED if not operator
     * ERR_NOSUCHNICK if user doesn't exist

5. **TOPIC** - View/Set Channel Topic
   - Usage: `TOPIC <channel> [topic]`
   - Test:
     * View current topic
     * Set new topic (if +t, only operators)
   - Error cases:
     * ERR_CHANOPRIVSNEEDED if +t and not operator
     * ERR_NOTONCHANNEL if not in channel

## Channel Modes
Test `MODE <channel> <modes> [parameters]`

1. **+i** (Invite-only)
   - Usage: `MODE <channel> +i`
   - Test: Only invited users can join

2. **+t** (Topic restriction)
   - Usage: `MODE <channel> +t`
   - Test: Only operators can change topic

3. **+k** (Channel key/password)
   - Usage: `MODE <channel> +k <key>`
   - Test: Key required to join

4. **+o** (Operator status)
   - Usage: `MODE <channel> +o <nickname>`
   - Test: Given user gets operator privileges

5. **+l** (User limit)
   - Usage: `MODE <channel> +l <limit>`
   - Test: Channel refuses joins when full

## Communication
1. **PRIVMSG** - Send Message
   - Usage: 
     * `PRIVMSG <channel> :<message>` (channel message)
     * `PRIVMSG <nickname> :<message>` (private message)
   - Test:
     * Channel messages
     * Private messages
     * Message formatting
   - Error cases:
     * ERR_CANNOTSENDTOCHAN if not in channel
     * ERR_NOSUCHNICK if user doesn't exist

2. **NAMES** - List Channel Users
   - Usage: `NAMES <channel>`
   - Test:
     * Shows all users in channel
     * Operators prefixed with @
   - Error cases:
     * ERR_NOSUCHCHANNEL if channel doesn't exist

## Testing Tools
1. **netcat (nc)**
   - Usage: `nc localhost 6667`
   - Good for:
     * Raw protocol testing
     * Seeing exact server responses
     * Testing partial messages

2. **irssi**
   - Usage: `/connect localhost 6667`
   - Good for:
     * Full client experience
     * Testing multiple connections
     * Channel operations

## Common Test Scenarios
1. **Authentication Flow**
```
PASS wrongpass
-> :ft_irc 464 * :Password incorrect
PASS correctpass
NICK alice
USER alice 0 * :Alice Smith
-> :ft_irc 001 alice :Welcome...
```

2. **Channel Creation & Modes**
```
JOIN #test
MODE #test +i
INVITE bob #test
-> bob can join despite +i
MODE #test +k secret
-> Others need password to join
```

3. **Operator Functions**
```
MODE #test +o charlie
-> charlie gets operator status
KICK #test bob :reason
-> bob is kicked from channel
```

4. **Error Handling**
```
JOIN #nonexistent
-> ERR_NOSUCHCHANNEL
PRIVMSG #test :message
-> ERR_CANNOTSENDTOCHAN (if not in channel)
```

## Evaluation Tips
1. Test with multiple simultaneous clients
2. Try combinations of channel modes
3. Test error cases and edge conditions
4. Verify proper message broadcasting
5. Check operator privileges
6. Ensure proper client disconnection handling 