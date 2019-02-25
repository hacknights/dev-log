# Side-Quests

``` text
/games/:name
	/start
		{ok}

	/:gameId
        /status
            {lives, gold, level, score, highscore, turn}
		
        /reputation
					{people, state, underworld}
		
        /quests
					[{questId, message, reward, expires}]
		
        /solve/:questId
					{ok, lives, gold, score, highscore, turn, message}
		
        /shop
					[{itemId, name, cost}]

					/buy/:itemId
						{ok, gold, turn}

					/sell/:itemId
						{ok, gold, turn}
```