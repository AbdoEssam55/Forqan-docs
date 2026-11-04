# Hifz direction (from-end)

## Next ayah

```csharp
SardCheckpointHelper.NextAyahInHifz(surahId, ayahNumber, hifzDirection);
```

1. Direction 1: ayah +1; last ayah → next surah ayah 1.
2. Direction 2: ayah +1; last ayah → **previous** surah ayah 1.
3. Do not decrement ayahs inside a surah (not 104:9 → 104:8).

Examples (direction 2):

- `114:1` → `114:2`
- `104:9` (Al-Humaza last) → `103:1` (Al-Asr 1)
- `103:1` → `103:2`

## Homework start cap

```csharp
HomeworkMemorizationWindowHelper.GetStartCap(student);
```

1. No reached → `StartOfHifz` (`1:1` or `114:1`).
2. Reached set → `NextAyahInHifz(reached)`.
3. Reject `From` after that cap (`HomeworkRangeBeforeReached`).

## Sard segment start

`ComputeCumulativeFrom` uses the same `NextAyahInHifz`. After a from-end checkpoint at last ayah of 78, next segment starts at `77:1`.
