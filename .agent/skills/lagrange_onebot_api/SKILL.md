---
name: Lagrange OneBot API (Group/User Management)
description: Reference guide for Lagrange OneBot API, focusing on Group and User management capabilities.
---

# Lagrange OneBot API Skill

This skill provides documentation and usage examples for the Lagrange OneBot implementation, specifically focusing on **Group Management** and **User/Friend Management**.

**Reference Documentation**: [Lagrange OneBot API](https://lagrange-onebot.apifox.cn/236981452e0)

## 1. Group Management (群管理)

### 1.1 Ban Member (单人禁言)
Mute a specific member in a group for a set duration.

- **Endpoint**: `set_group_ban`
- **Parameters**:
  - `group_id` (int64, Required): Target group ID.
  - `user_id` (int64, Required): Target user ID to ban.
  - `duration` (int32, Optional): Ban duration in seconds. Default: 30 minutes (1800). Set to 0 to unban.

**Example Payload**:
```json
{
    "action": "set_group_ban",
    "params": {
        "group_id": 123456789,
        "user_id": 987654321,
        "duration": 600  // 10 minutes
    }
}
```

### 1.2 Whole Group Ban (全员禁言)
Turn on/off whole group mute.

- **Endpoint**: `set_group_whole_ban`
- **Parameters**:
  - `group_id` (int64, Required): Target group ID.
  - `enable` (bool, Optional): `true` to enable, `false` to disable. Default: `true`.

**Example Payload**:
```json
{
    "action": "set_group_whole_ban",
    "params": {
        "group_id": 123456789,
        "enable": true
    }
}
```

### 1.3 Kick Member (踢人)
Kick a member from a group.

- **Endpoint**: `set_group_kick`
- **Parameters**:
  - `group_id` (int64, Required): Target group ID.
  - `user_id` (int64, Required): Target user ID to kick.
  - `reject_add_request` (bool, Optional): If `true`, block future join requests from this user. Default: `false`.

**Example Payload**:
```json
{
    "action": "set_group_kick",
    "params": {
        "group_id": 123456789,
        "user_id": 987654321,
        "reject_add_request": false
    }
}
```

### 1.4 Set Group Admin (设置管理员)
Set or remove a group administrator.

- **Endpoint**: `set_group_admin`
- **Parameters**:
  - `group_id` (int64, Required): Target group ID.
  - `user_id` (int64, Required): Target user ID.
  - `enable` (bool, Optional): `true` to set as admin, `false` to remove. Default: `true`.

**Example Payload**:
```json
{
    "action": "set_group_admin",
    "params": {
        "group_id": 123456789,
        "user_id": 987654321,
        "enable": true
    }
}
```

### 1.5 Set Group Card (设置群名片)
Modify a user's nickname in the group.

- **Endpoint**: `set_group_card`
- **Parameters**:
  - `group_id` (int64, Required): Target group ID.
  - `user_id` (int64, Required): Target user ID.
  - `card` (string, Optional): New nickname. Empty string clears the card.

**Example Payload**:
```json
{
    "action": "set_group_card",
    "params": {
        "group_id": 123456789,
        "user_id": 987654321,
        "card": "New Nickname"
    }
}
```

### 1.6 Set Special Title (设置群头衔)
Set a specific title for a group member (Robot must be Owner).

- **Endpoint**: `set_group_special_title`
- **Parameters**:
  - `group_id` (int64, Required): Target group ID.
  - `user_id` (int64, Required): Target user ID.
  - `special_title` (string, Optional): The title to set.
  - `duration` (int32, Optional): Validity duration in seconds. -1 for permanent.

**Example Payload**:
```json
{
    "action": "set_group_special_title",
    "params": {
        "group_id": 123456789,
        "user_id": 987654321,
        "special_title": "Gold Member",
        "duration": -1
    }
}
```

---

## 2. User & Request Management (用户与请求管理)

### 2.1 Handle Friend Request (处理加好友请求)
Approve or deny a friend request.

- **Endpoint**: `set_friend_add_request`
- **Parameters**:
  - `flag` (string, Required): The `flag` identifier from the request event.
  - `approve` (bool, Optional): `true` to approve, `false` to deny. Default: `true`.
  - `remark` (string, Optional): Remark name for the new friend (only if approved).

**Example Payload**:
```json
{
    "action": "set_friend_add_request",
    "params": {
        "flag": "request_flag_string",
        "approve": true,
        "remark": "My Friend"
    }
}
```

### 2.2 Handle Group Request (处理加群/邀请请求)
Approve or deny a group join/invite request.

- **Endpoint**: `set_group_add_request`
- **Parameters**:
  - `flag` (string, Required): The `flag` identifier from the request event.
  - `sub_type` (string, Required): `add` (join request) or `invite` (invitation).
  - `approve` (bool, Optional): `true` to approve, `false` to deny. Default: `true`.
  - `reason` (string, Optional): Reason for denial (only if denied).

**Example Payload**:
```json
{
    "action": "set_group_add_request",
    "params": {
        "flag": "request_flag_string",
        "sub_type": "add",
        "approve": true
    }
}
```

### 2.3 Delete Friend (删除好友)
Remove a user from friend list.

- **Endpoint**: `delete_friend`
- **Parameters**:
  - `user_id` (int64, Required): Target user ID.

**Example Payload**:
```json
{
    "action": "delete_friend",
    "params": {
        "user_id": 987654321
    }
}
```

---

## 3. QBot Implementation Example

Here is how you would use these APIs within a QBot module (`modules/your_module/module.py`).

```python
import json
from core.base_module import BaseModule, ModuleContext, ModuleResponse

class GroupManagerModule(BaseModule):
    # ... init ...

    async def handle(self, message: str, context: ModuleContext) -> Optional[ModuleResponse]:
        # Example: Ban a user if they say "ban me"
        if message == "ban me":
             # Construct the OneBot API payload
            ban_payload = {
                "action": "set_group_ban",
                "params": {
                    "group_id": context.group_id,
                    "user_id": context.user_id,
                    "duration": 60  # 1 minute ban
                },
                "echo": f"ban_{context.user_id}" # Optional echo for tracking response
            }
            
            # Send the payload via WebSocket
            await context.ws.send_text(json.dumps(ban_payload))
            
            return ModuleResponse(content="You have been banned for 1 minute.")
            
        return None
```
