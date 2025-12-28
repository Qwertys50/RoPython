import aiohttp
import asyncio

import json
import inspect

#def get_data() -> dict: 
#    with open("config.json") as f: return json.load(f)


class IsPlaying:
    def __init__(self, user_id: int, token: str):

        self.user_id: int = user_id
        self.token: str = token
        self.headers: dict = {
            "Cookie": f"{self.token};"
        }

        self._event_callback = None
        self.__type: None | int = None
        self.__game: None | str = None

    def event_game(self):

        def decorator(func):
            self._event_callback = func
            return func
        return decorator

    async def _get_status(self, session: aiohttp.ClientSession):

        async with session.post(
            "https://presence.roblox.com/v1/presence/users",
            json={"userIds": [self.user_id]},
            headers=self.headers

        ) as resp: return resp.status, await resp.json()

    async def start(self):

        async with aiohttp.ClientSession() as session:

            while True:

                status, json_data = await self._get_status(session)

                if status in [200, 201]:

                    presence = json_data["userPresences"][0]
                    curr_type = presence["userPresenceType"]
                    curr_game = presence.get("lastLocation", "")

                    server_id = presence.get("gameId")
                    userId = presence.get("userId")
                    placeId = presence.get("placeId")

                    if self.__type in [0, 1] and curr_type == 2:
                        
                        if self._event_callback:
                            await self._call_event(
                                type="join",
                                game=curr_game,
                                server_id=server_id,
                                userId=userId,
                                placeId=placeId
                            )


                    elif self.__type == 2 and curr_type in [0, 1]:

                        if self._event_callback:

                            await self._call_event(
                                type="leave",
                                game=self.__game,
                                server_id=server_id,
                                userId=userId,
                                placeId=placeId
                            )


                    self.__type = curr_type
                    self.__game = curr_game

                    await asyncio.sleep(0)

                elif status == 429: await asyncio.sleep(1)

    async def _call_event(self, **kwargs):
        sig = inspect.signature(self._event_callback)

        filtered_kwargs = {
            name: value
            for name, value in kwargs.items()
            if name in sig.parameters
        }

        result = self._event_callback(**filtered_kwargs)
        if asyncio.iscoroutine(result):
            await result

#data = get_data()
data = {"userId":1234567890, "token":""}

plaing = IsPlaying(data["userId"], data["token"]) 

@plaing.event_game() 
def main(type, game, userId): 
    print(type, userId)

asyncio.run(plaing.start())
