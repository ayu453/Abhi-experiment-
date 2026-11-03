# Relayless — Architecture & Prototype Notes

## Product goal
A browser-based, one-to-one messaging prototype. Each browser profile creates a local device identity and stores its own chat history locally. There is no application backend, central message database, hosted account system, or message relay.

## Prototype 0.2
- Static single-page app: `index.html`.
- Local account ID and display name stored in IndexedDB.
- Messages and the last conversation peer are stored in IndexedDB on that device.
- WebRTC DataChannel transports messages directly between connected browsers.
- WebRTC uses its encrypted transport. This is not yet a reviewed end-to-end identity/authentication protocol.
- STUN is used for NAT discovery. It does not carry chat messages. Some NAT/firewall combinations will prevent direct connectivity without TURN.
- Signaling is manual: exchange an offer package and an answer package by copy/paste. No signaling server is used.
- Sender-side messages stay in a durable local queue until a delivery acknowledgement is received.
- On a newly established connection, the sender retries every message not marked delivered. The receiver deduplicates by message ID and acknowledges duplicate deliveries without creating duplicate chat bubbles.
- After refresh or a network drop, users can create a fresh invite and repeat the manual connection exchange. The app remembers the last peer and local conversation.

## Honest limitations
1. Offline queue is sender-side only. If the sender closes/offlines, no always-on relay exists to deliver messages. The sender must come online again and reconnect.
2. A WebRTC connection does not automatically survive browser restarts. Users must establish a new connection by exchanging a fresh invite and reply.
3. Browser storage is per browser profile/device, not a cloud account. Clearing site data or losing the device can erase history. Export backups regularly.
4. The account ID is a device identity, not a verified global username. There is no server-side identity directory or password reset.
5. STUN and signaling are different: STUN assists network path discovery; manual signaling exchanges connection descriptions. TURN relay is intentionally not configured because it would relay traffic through infrastructure.
6. A message marked "sent" is not yet confirmed. Only a received acknowledgement changes it to "delivered".
7. This prototype is not production-ready. It needs a security review, cryptographic identity verification, robust reconnect UX, authenticated peer identity, backup import/restore, multi-device account design, rate limits, and abuse controls.

## Data model
- `meta`: `accountId`, `name`, `lastPeerId`, `lastPeerName`
- `messages`: `id`, `peerId`, `from`, `text`, `time`, `status`
- Each browser writes only to its own local IndexedDB.

## Connection sequence
1. Device A creates a WebRTC offer and copies the package.
2. Device B pastes it, creates an answer, and sends the answer package back.
3. Device A pastes the answer; ICE negotiation completes if the network allows direct connectivity.
4. A data channel opens. Peers exchange a hello packet and messages.
5. Receiver saves each message locally and sends an acknowledgement. Sender marks it delivered.
6. If an acknowledgement is lost, sender retries after the next connection; receiver deduplicates by message ID.

## Next engineering milestones
- Add optional QR pairing without sending invite contents to a third-party QR API.
- Add encrypted local backups and import/restore.
- Add a cryptographic account keypair and verify peer fingerprints out-of-band.
- Add attachment transfer with limits and progress.
- Decide whether strict no-server policy or reliable offline delivery is the priority. Reliable delivery while both users are offline requires some always-available relay/store-and-forward infrastructure.
