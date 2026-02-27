<svelte:head>
  <title>Contact me</title>
</svelte:head>
<h1>Contact</h1>
<form action="mailto:hannach8@mit.edu" method="get">
    <label>
        Email: 
        <input name="email" type="email"/>
    </label>
    <label>
        Subject:
        <input name="subject"/>
    </label>
    <label>
        Message:
        <textarea name="body"></textarea>
    </label>
    <button>Send</button>
</form>