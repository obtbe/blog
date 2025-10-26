---
title: "Contact"
date: 2025-04-05T10:00:00+05:30
type: "page"
---

Feel free to get in touch about anything you see here. I am zobtbe at gmail, and you can find me on [Telegram](https://t.me/obtbe).

I maintain a mailing list where I announce new data and programming-focused articles and resources. If you’re interested in receiving these notifications, you can sign up below. I don’t send emails often, and you can unsubscribe at any time.



<form id="substack-form" style="margin-top:1rem;">
  <input 
    type="email" 
    id="substack-email" 
    placeholder="Enter your email" 
    required 
    style="padding:0.8rem; width:70%; max-width:300px; border:1px solid #CCC;">
  <button 
    type="submit" 
    style="padding:0.8rem 1.2rem; background:#007acc; color:white; border:none; cursor:pointer;">
    Subscribe
  </button>
</form>

<p id="substack-result" style="margin-top:0.5rem; font-size:0.9rem;"></p>

<script>
document.getElementById('substack-form').addEventListener('submit', async function (e) {
  e.preventDefault();
  const email = document.getElementById('substack-email').value;
  const result = document.getElementById('substack-result');

  try {
    const res = await fetch('https://obtbe.substack.com/api/v1/free', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email })
    });

    if (res.status === 200) {
      result.textContent = "Check your email to confirm your subscription.";
    } else {
      result.textContent = "Something went wrong. Try again.";
    }
  } catch (err) {
    result.textContent = "Network error. Try again.";
  }
});
</script>

