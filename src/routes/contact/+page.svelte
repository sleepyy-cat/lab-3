<svelte:head>
  <title>Contact me</title>
</svelte:head>
<nav>
    <a href="./">Home</a>
    <a href="projects">Projects</a>
    <a href="." class="current">Contact</a>
    <a href="resume">Resume</a>
    <a href="https://github.com/sleepyy-cat" target="_blank">Github</a>
</nav>
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