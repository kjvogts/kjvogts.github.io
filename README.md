<!DOCTYPE html>
<html>
<head>
  <title>Run Like the Winded</title>
  <script src="https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2"></script>

  <style>
    body {
      margin:0;
      font-family: system-ui;
      background:#140a26;
      color:white;
      display:flex;
      justify-content:center;
      align-items:center;
      min-height:100vh;
    }

    .card {
      width: 350px;
      background: rgba(255,255,255,0.06);
      padding: 20px;
      border-radius: 12px;
    }

    input, button {
      width:100%;
      padding:10px;
      margin-top:10px;
      border-radius:8px;
      border:none;
    }

    button {
      background:#f6e27a;
      color:#140a26;
      font-weight:bold;
      cursor:pointer;
    }

    .hidden { display:none; }
  </style>
</head>

<body>

<div class="card" id="authBox">
  <h2>Run Like the Winded</h2>

  <input id="email" placeholder="email">
  <input id="password" type="password" placeholder="password">

  <button onclick="signUp()">Create Account</button>
  <button onclick="login()">Login</button>
</div>

<div class="card hidden" id="appBox">
  <h2>Log a Run</h2>

  <input id="distance" placeholder="Distance (km)">
  <input id="time" placeholder="Time (minutes)">
  <input id="notes" placeholder="Notes">

  <button onclick="saveRun()">Save Run</button>

  <h3>Your Runs</h3>
  <div id="runs"></div>

  <button onclick="logout()">Logout</button>
</div>

<script>
const supabase = window.supabase.createClient(
  "YOUR_SUPABASE_URL",
  "YOUR_SUPABASE_ANON_KEY"
);

async function signUp(){
  const email = document.getElementById("email").value;
  const password = document.getElementById("password").value;

  const { error } = await supabase.auth.signUp({ email, password });

  if(error) return alert(error.message);
  alert("Account created! Now login.");
}

async function login(){
  const email = document.getElementById("email").value;
  const password = document.getElementById("password").value;

  const { error } = await supabase.auth.signInWithPassword({ email, password });

  if(error) return alert(error.message);

  loadApp();
}

async function loadApp(){
  document.getElementById("authBox").classList.add("hidden");
  document.getElementById("appBox").classList.remove("hidden");
  loadRuns();
}

async function saveRun(){
  const distance = document.getElementById("distance").value;
  const time = document.getElementById("time").value;
  const notes = document.getElementById("notes").value;

  const user = await supabase.auth.getUser();

  await supabase.from("runs").insert([{
    user_id: user.data.user.id,
    distance,
    time_minutes: time,
    notes
  }]);

  loadRuns();
}

async function loadRuns(){
  const user = await supabase.auth.getUser();

  const { data } = await supabase
    .from("runs")
    .select("*")
    .eq("user_id", user.data.user.id)
    .order("created_at", { ascending:false });

  document.getElementById("runs").innerHTML =
    data.map(r =>
      `<div style="margin-top:10px;padding:8px;background:rgba(255,255,255,0.05);border-radius:8px">
        ${r.distance} km — ${r.time_minutes} min<br>
        ${r.notes}
      </div>`
    ).join("");
}

async function logout(){
  await supabase.auth.signOut();
  location.reload();
}
</script>

</body>
</html>
