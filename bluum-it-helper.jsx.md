# bluum-it-helper.jsx

import { useState, useRef, useEffect } from "react";

const HANDBOOK_SYSTEM_PROMPT = `You are "IT Helper," a friendly, patient AI assistant for non-technical school staff at a small education nonprofit. Your job is to walk users through common IT issues step by step BEFORE they submit a help desk ticket.

IMPORTANT RULES:
- Use simple, non-technical language. These are teachers, administrators, and office staff — not IT professionals.
- Give ONE step at a time. Wait for the user to confirm before moving on.
- Be warm, encouraging, and patient. Never make someone feel dumb.
- If the issue cannot be resolved through your guidance, help them write a clear ticket summary.
- Always ask clarifying questions when the problem is vague.
- Format steps clearly with numbers or bullet points when appropriate.

You have deep knowledge of the following IT support areas from the help desk handbook:

--- WINDOWS BASICS ---
- Windows 11 Settings, Device Manager, Services, Event Viewer, Control Panel
- Command Prompt and PowerShell basics
- Finding computer name (hostname), IP address (ipconfig), renaming PC, creating/removing local accounts

--- PASSWORD RESETS ---
Standard procedure:
1. Verify the user's identity
2. The IT team uses Microsoft Entra Admin Center to locate the account
3. Password is reset and user must change it at next sign-in
4. Account is unlocked if necessary
Common causes: Caps Lock on, expired password, disabled account, locked account, MFA issue
SELF-HELP: Before submitting a ticket, have the user check Caps Lock, try typing password in Notepad to verify, check if they recently changed their password, and try on a different device.

--- MICROSOFT ENTRA ID ---
- User account lifecycle: create, disable, delete, restore
- License assignment, group management, MFA reset
- Sign-in log review for troubleshooting

--- MICROSOFT 365 ---
- Outlook: mailbox issues, activation problems, shared mailboxes
- Teams: microphone, camera, meeting issues
- OneDrive: sync troubleshooting
- Office: activation and licensing errors
SELF-HELP for Teams: Check browser permissions for mic/camera, try Teams web vs desktop app, restart the app, check Windows sound settings.
SELF-HELP for OneDrive: Check the sync icon in system tray, pause and resume sync, sign out and back in.

--- PRINTERS ---
- Installing network printers via IP address
- Driver installation, test pages
- Troubleshooting: offline printers, paper jams, stuck queues, driver issues
SELF-HELP steps:
1. Check if printer shows "Offline" — toggle "Use Printer Offline" off
2. Check for paper jams (open all trays and doors)
3. Restart the printer (power off, wait 30 seconds, power on)
4. On Windows: open Services (services.msc) → find Print Spooler → Restart
5. Clear print queue: Settings → Printers & Scanners → select printer → Open Queue → Cancel All

--- WI-FI TROUBLESHOOTING ---
- First determine: is it just you or multiple people?
SELF-HELP steps:
1. Toggle Wi-Fi off and back on
2. Forget the network and reconnect (you'll need the password)
3. Restart your device
4. If on Windows, open Command Prompt and try: ipconfig /release then ipconfig /renew then ipconfig /flushdns
5. Try ping google.com to test connectivity
If multiple people are affected, it's likely a network issue — submit a ticket immediately.

--- PROJECTOR / DISPLAY ISSUES ---
- Check cable connections (HDMI, VGA, USB-C)
- Try Win+P to switch display mode (Duplicate, Extend)
- Restart the projector
- Try a different cable
- Check input source on the projector remote

--- CHROMEBOOK WI-FI ---
- Restart the Chromebook
- Forget and rejoin the network
- Check if other devices connect to the same network
- Check that the correct network is selected (school vs guest)

--- SLOW INTERNET ---
- Close unnecessary browser tabs and applications
- Restart the device
- Test on another device to isolate the issue
- If widespread, report to IT with details about when it started

--- TICKET DOCUMENTATION ---
A good ticket includes:
- What the problem is (specific error messages if any)
- When it started
- What device/application is affected
- What you've already tried
- How many people are affected
Example: "Teacher unable to print from Room 204 laptop to hallway printer. Printer shows offline. Restarted printer and cleared queue — still offline. Started this morning. Only affecting this one printer."

--- TROUBLESHOOTING MINDSET ---
Always help the user think through:
- What changed recently?
- When exactly did it start?
- Is it just you or everyone?
- Is there an error message? (Ask them to take a screenshot or read it exactly)
- What have you already tried?

--- KEYBOARD SHORTCUTS (helpful to share) ---
- Win+I → Settings
- Ctrl+Shift+Esc → Task Manager
- Win+R → Run dialog
- Win+E → File Explorer

When you CANNOT resolve the issue, generate a ticket summary the user can copy/paste, formatted like:
**Issue:** [brief description]
**Device:** [if known]
**Steps Tried:** [what was attempted]
**Details:** [any error messages, when it started, who's affected]

Keep responses concise. School staff are busy people.`;

const QUICK_ACTIONS = [
  { id: "password", icon: "🔑", label: "Password Problem", prompt: "I can't log in — I think my password isn't working." },
  { id: "printer", icon: "🖨️", label: "Printer Issue", prompt: "My printer isn't working. It might be offline or nothing is printing." },
  { id: "wifi", icon: "📶", label: "Wi-Fi Trouble", prompt: "I'm having trouble connecting to the Wi-Fi." },
  { id: "teams", icon: "💬", label: "Teams / Camera", prompt: "I'm having issues with Microsoft Teams — my camera or microphone might not be working." },
  { id: "projector", icon: "📽️", label: "Projector / Display", prompt: "My projector or external display isn't showing anything." },
  { id: "slow", icon: "🐌", label: "Slow Computer", prompt: "My computer or internet is running really slowly today." },
  { id: "outlook", icon: "📧", label: "Email / Outlook", prompt: "I'm having a problem with my email in Outlook." },
  { id: "onedrive", icon: "☁️", label: "OneDrive Sync", prompt: "My OneDrive files aren't syncing properly." },
];

function TypingIndicator() {
  return (
    <div style={{ display: "flex", gap: 5, padding: "12px 16px", alignItems: "center" }}>
      {[0, 1, 2].map(i => (
        <div key={i} style={{
          width: 8, height: 8, borderRadius: "50%", background: "#7BA99A",
          animation: `bounce 1.2s ease-in-out ${i * 0.15}s infinite`,
        }} />
      ))}
    </div>
  );
}

function MessageBubble({ role, content }) {
  const isUser = role === "user";
  return (
    <div style={{
      display: "flex", justifyContent: isUser ? "flex-end" : "flex-start",
      marginBottom: 12, paddingInline: 4,
    }}>
      {!isUser && (
        <div style={{
          width: 32, height: 32, borderRadius: "50%", background: "#1B6B5A",
          display: "flex", alignItems: "center", justifyContent: "center",
          fontSize: 15, flexShrink: 0, marginRight: 10, marginTop: 4,
          color: "#fff", fontWeight: 700,
        }}>IT</div>
      )}
      <div style={{
        maxWidth: "78%", padding: "11px 16px", borderRadius: 16,
        background: isUser ? "#1B6B5A" : "#EDF3F0",
        color: isUser ? "#fff" : "#1A2B2A",
        fontSize: 14.5, lineHeight: 1.55,
        borderBottomRightRadius: isUser ? 4 : 16,
        borderBottomLeftRadius: isUser ? 16 : 4,
        whiteSpace: "pre-wrap", wordBreak: "break-word",
      }}>
        {content.split(/(\*\*.*?\*\*)/).map((part, i) => {
          if (part.startsWith("**") && part.endsWith("**")) {
            return <strong key={i}>{part.slice(2, -2)}</strong>;
          }
          return part;
        })}
      </div>
    </div>
  );
}

export default function BluumITHelper() {
  const [messages, setMessages] = useState([]);
  const [input, setInput] = useState("");
  const [loading, setLoading] = useState(false);
  const [started, setStarted] = useState(false);
  const [ticketMode, setTicketMode] = useState(false);
  const scrollRef = useRef(null);
  const inputRef = useRef(null);

  useEffect(() => {
    if (scrollRef.current) {
      scrollRef.current.scrollTop = scrollRef.current.scrollHeight;
    }
  }, [messages, loading]);

  const sendMessage = async (text) => {
    if (!text.trim() || loading) return;
    const userMsg = { role: "user", content: text.trim() };
    const updatedMessages = [...messages, userMsg];
    setMessages(updatedMessages);
    setInput("");
    setStarted(true);
    setLoading(true);

    try {
      const apiMessages = updatedMessages.map(m => ({
        role: m.role, content: m.content,
      }));

      const res = await fetch("https://api.anthropic.com/v1/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({
          model: "claude-sonnet-4-6",
          max_tokens: 1000,
          system: HANDBOOK_SYSTEM_PROMPT,
          messages: apiMessages,
        }),
      });

      const data = await res.json();
      const reply = data.content
        ?.filter(b => b.type === "text")
        .map(b => b.text)
        .join("\n") || "I'm sorry, I had trouble responding. Could you try again?";

      setMessages(prev => [...prev, { role: "assistant", content: reply }]);
    } catch (err) {
      setMessages(prev => [...prev, {
        role: "assistant",
        content: "Hmm, I'm having a connection issue right now. Please check your internet and try again, or go ahead and submit a ticket directly.",
      }]);
    } finally {
      setLoading(false);
    }
  };

  const handleQuickAction = (action) => {
    sendMessage(action.prompt);
  };

  const handleSubmit = (e) => {
    e.preventDefault();
    sendMessage(input);
  };

  const handleGenerateTicket = () => {
    sendMessage("Based on everything we've discussed, please generate a formatted help desk ticket summary I can copy and send to IT.");
    setTicketMode(true);
  };

  const handleReset = () => {
    setMessages([]);
    setStarted(false);
    setTicketMode(false);
    setInput("");
  };

  return (
    <div style={{
      height: "100vh", width: "100%", display: "flex", flexDirection: "column",
      fontFamily: "'Inter', 'Segoe UI', system-ui, -apple-system, sans-serif",
      background: "#F4F8F6", color: "#1A2B2A", overflow: "hidden",
    }}>
      <style>{`
        @keyframes bounce {
          0%, 60%, 100% { transform: translateY(0); }
          30% { transform: translateY(-6px); }
        }
        @keyframes fadeUp {
          from { opacity: 0; transform: translateY(12px); }
          to { opacity: 1; transform: translateY(0); }
        }
        .qa-btn {
          display: flex; align-items: center; gap: 10px;
          padding: 12px 16px; border: 1.5px solid #C8D8D1;
          border-radius: 12px; background: #fff; cursor: pointer;
          font-size: 14px; color: #1A2B2A; text-align: left;
          transition: all 0.15s ease;
          font-family: inherit; width: 100%;
        }
        .qa-btn:hover { border-color: #1B6B5A; background: #EDF3F0; transform: translateY(-1px); box-shadow: 0 2px 8px rgba(27,107,90,0.08); }
        .qa-btn:active { transform: translateY(0); }
        .send-btn {
          background: #1B6B5A; color: #fff; border: none;
          border-radius: 12px; padding: 10px 20px;
          font-size: 14px; font-weight: 600; cursor: pointer;
          transition: background 0.15s; font-family: inherit;
        }
        .send-btn:hover { background: #155A4B; }
        .send-btn:disabled { background: #A5C4BA; cursor: not-allowed; }
        .action-link {
          background: none; border: 1px solid #C8D8D1; color: #1B6B5A;
          padding: 8px 16px; border-radius: 10px; font-size: 13px;
          cursor: pointer; font-weight: 500; transition: all 0.15s;
          font-family: inherit;
        }
        .action-link:hover { background: #EDF3F0; border-color: #1B6B5A; }
        input::placeholder { color: #8FA59C; }
      `}</style>

      {/* Header */}
      <div style={{
        background: "#1B6B5A", color: "#fff", padding: "16px 24px",
        display: "flex", alignItems: "center", justifyContent: "space-between",
        flexShrink: 0,
      }}>
        <div style={{ display: "flex", alignItems: "center", gap: 12 }}>
          <div style={{
            width: 38, height: 38, borderRadius: 10, background: "rgba(255,255,255,0.18)",
            display: "flex", alignItems: "center", justifyContent: "center",
            fontSize: 20, fontWeight: 800,
          }}>🛠️</div>
          <div>
            <div style={{ fontWeight: 700, fontSize: 17, letterSpacing: "-0.02em" }}>IT Helper</div>
            <div style={{ fontSize: 12, opacity: 0.8 }}>Troubleshoot before you ticket</div>
          </div>
        </div>
        {started && (
          <div style={{ display: "flex", gap: 8 }}>
            <button className="action-link" onClick={handleGenerateTicket}
              style={{ color: "#fff", borderColor: "rgba(255,255,255,0.35)" }}>
              📋 Generate Ticket
            </button>
            <button className="action-link" onClick={handleReset}
              style={{ color: "#fff", borderColor: "rgba(255,255,255,0.35)" }}>
              ↺ New Issue
            </button>
          </div>
        )}
      </div>

      {/* Chat Area */}
      <div ref={scrollRef} style={{
        flex: 1, overflowY: "auto", padding: "20px 16px",
        display: "flex", flexDirection: "column",
      }}>
        {!started ? (
          <div style={{ animation: "fadeUp 0.4s ease", maxWidth: 600, margin: "0 auto", width: "100%" }}>
            <div style={{ textAlign: "center", marginBottom: 28 }}>
              <div style={{
                width: 64, height: 64, borderRadius: 18, background: "#1B6B5A",
                display: "inline-flex", alignItems: "center", justifyContent: "center",
                fontSize: 32, marginBottom: 14,
              }}>🛠️</div>
              <h2 style={{ margin: 0, fontSize: 22, fontWeight: 700, color: "#1A2B2A" }}>
                Hi there! What's going on?
              </h2>
              <p style={{ margin: "8px 0 0", color: "#5A7A70", fontSize: 14.5, lineHeight: 1.5 }}>
                I'll walk you through fixing common tech issues step by step.<br />
                Pick a category below or describe your problem.
              </p>
            </div>

            <div style={{
              display: "grid", gridTemplateColumns: "repeat(auto-fill, minmax(220px, 1fr))",
              gap: 10,
            }}>
              {QUICK_ACTIONS.map(action => (
                <button key={action.id} className="qa-btn" onClick={() => handleQuickAction(action)}>
                  <span style={{ fontSize: 22, flexShrink: 0 }}>{action.icon}</span>
                  <span style={{ fontWeight: 500 }}>{action.label}</span>
                </button>
              ))}
            </div>

            <p style={{
              textAlign: "center", color: "#8FA59C", fontSize: 12.5,
              marginTop: 24, lineHeight: 1.6,
            }}>
              Powered by the Bluum Help Desk Handbook · Built for school staff
            </p>
          </div>
        ) : (
          <>
            {messages.map((msg, i) => (
              <MessageBubble key={i} role={msg.role} content={msg.content} />
            ))}
            {loading && (
              <div style={{ display: "flex", alignItems: "flex-start", gap: 10, marginBottom: 12 }}>
                <div style={{
                  width: 32, height: 32, borderRadius: "50%", background: "#1B6B5A",
                  display: "flex", alignItems: "center", justifyContent: "center",
                  fontSize: 15, color: "#fff", fontWeight: 700, flexShrink: 0,
                }}>IT</div>
                <div style={{
                  background: "#EDF3F0", borderRadius: 16,
                  borderBottomLeftRadius: 4,
                }}>
                  <TypingIndicator />
                </div>
              </div>
            )}
          </>
        )}
      </div>

      {/* Input */}
      <div style={{
        padding: "14px 16px 18px", background: "#fff",
        borderTop: "1px solid #D4DDD9", flexShrink: 0,
      }}>
        <form onSubmit={handleSubmit} style={{
          display: "flex", gap: 10, maxWidth: 700, margin: "0 auto",
        }}>
          <input
            ref={inputRef}
            type="text"
            value={input}
            onChange={e => setInput(e.target.value)}
            placeholder={started ? "Describe what's happening…" : "Or type your issue here…"}
            disabled={loading}
            style={{
              flex: 1, padding: "12px 16px", borderRadius: 12,
              border: "1.5px solid #C8D8D1", fontSize: 14.5,
              outline: "none", fontFamily: "inherit", color: "#1A2B2A",
              background: loading ? "#F4F8F6" : "#fff",
              transition: "border-color 0.15s",
            }}
            onFocus={e => e.target.style.borderColor = "#1B6B5A"}
            onBlur={e => e.target.style.borderColor = "#C8D8D1"}
          />
          <button type="submit" className="send-btn" disabled={!input.trim() || loading}>
            {loading ? "…" : "Send"}
          </button>
        </form>
        <p style={{
          textAlign: "center", fontSize: 11.5, color: "#A5B8B0",
          margin: "10px 0 0", lineHeight: 1.4,
        }}>
          This tool helps with common fixes. For emergencies (network down, security incident), contact IT directly.
        </p>
      </div>
    </div>
  );
}


---

*Created: 7/26/2026, 5:53:08 PM*
*Document ID: eaf0577c-10cd-441f-9346-c7d194fbccd7*