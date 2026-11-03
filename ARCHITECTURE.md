# Relayless — Architecture & Prototype Notes

## Product goal
A browser-based, one-to-one messaging prototype. Each browser profile creates a local account identity and stores its own chat history locally. No application backend, central message database, hosted account system, or message relay is included.

## Current prototype
- Static single-page app: `index.html`
- Local account ID and display name stored in IndexedDB.
- Messages stored in IndexedDB on the device that sends or receives them.
- WebRTC DataChannel transports messages directly between connected browsers.
- WebRTC uses DTLS/SRTP-class transport security for the peer connection; this is not yet a reviewed end-to-end identity/authentication protocol.
- STUN is used for NAT discovery. It does not carry chat messages. Some NAT/firewall combinations will prevent direct connectivity without TURN.
- Signaling is manual: exchange an offer package and an answer package by copy/paste. No signaling server is used.
- Sender-side pending messages remain in local IndexedDB. When the same peer connection is online, pending messages can be sent and acknowledged.

## Honest limitations
1. A sender-side pending queue is not an offline delivery service. If the sender closes/offlines before the recipient reconnects, no always-on relay exists to deliver it. The sender must come online again and reconnect.
2. A WebRTC connection does not automatically survive browser restarts. The peers must establish a new connection.
3. Browser storage is per browser profile/device, not a cloud account. Clearing site data or losing the device can erase history. Export backups regularly.
4. A local account ID is a device identity, not a verified global username. There is no server-side identity directory or password reset.
5. STUN and signaling are different: STUN assists network path discovery; manual signaling exchanges connection descriptions. TURN relay is intentionally not configured because it would relay traffic through infrastructure.
6. This prototype is not production-ready. It needs security review, robust reconnect/retry logic, message deduplication, authenticated peer identity, import/restore, multi-device account design, and abuse controls.

## Data model
- `meta`: `accountId`, `name`
- `messages`: `id`, `peerId`, `from`, `text`, `time`, `status`
- Each browser writes only to its own local IndexedDB.

## Connection sequence
1. Device A creates a WebRTC offer and copies the package.
2. Device B pastes it, creates an answer, and sends the answer package back.
3. Device A pastes the answer; ICE negotiation completes if the network allows direct connectivity.
4. A data channel opens. Peers exchange a hello packet and messages.
5. Receiver saves each message locally and sends an acknowledgement. Sender changes status to delivered.

## Next engineering milestones
- Add encrypted local backups and import/restore.
- Add robust message IDs, retry queue, deduplication, and reconnect UX.
- Add cryptographic account keypair and verify peer fingerprints out-of-band.
- Add attachment transfer with limits and progress.
- Decide whether strict no-server policy or reliable offline delivery is the priority. Reliable delivery while both users are offline requires some always-available relay/store-and-forward infrastructure.
