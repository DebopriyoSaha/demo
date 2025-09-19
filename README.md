<div className="flex items-center gap-2">
  <Button asChild>
    <Link to="/new">Create New Entry</Link>
  </Button>
  <Button variant="ghost" asChild className="bg-white/60 hover:bg-white">
    <Link to="/profiles">Switch Profile</Link>
  </Button>
  <Button
    variant="ghost"
    onClick={() => {
      logout();
      navigate("/login", { replace: true });
    }}
  >
    Logout
  </Button>
</div>
