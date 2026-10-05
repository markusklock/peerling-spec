# 2026-10-05 — Phone backup instead of device linking

Kind: design conversation. The LLM had proposed details for D-0016 (Q-044):
spectating, sharing links, device linking (short code + PAKE, one device
playing at a time) and the world feed.

## Designer's answer (verbatim)

> 1, 2 and 4 approved.
> For 3 we should redesign. Perhaps just use the QR code and scan with mobile device as backup/export function? It triggers your phone to fetch the player save so that you have a copy you can bring with you, that can be imported on another computer or if you clear your browser cache. You are not meant to be able to play the game on your phone.

## Summary of what was decided

- Details of spectating, sharing links and the world feed approved.
- Device linking is replaced by a **phone backup**: the computer shows a QR
  code; scanning it with a phone makes the phone fetch the player's save, so
  the player carries a copy that can be imported on another computer or after
  clearing browser data. The phone is not for playing.
