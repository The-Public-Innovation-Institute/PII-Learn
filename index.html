import { createClient } from 'https://esm.sh/@supabase/supabase-js@2'

const corsHeaders = {
  'Access-Control-Allow-Origin': '*',
  'Access-Control-Allow-Headers': 'authorization, x-client-info, apikey, content-type',
}

Deno.serve(async (req) => {
  if (req.method === 'OPTIONS') return new Response('ok', { headers: corsHeaders })

  try {
    // Verify the caller is an admin
    const authHeader = req.headers.get('Authorization')
    if (!authHeader) throw new Error('Missing authorization header')

    const callerClient = createClient(
      Deno.env.get('adtgtzblbneqedkysurk.supabase.co')!,
      Deno.env.get('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFkdGd0emJsYm5lcWVka3lzdXJrIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODI1NjM2MTcsImV4cCI6MjA5ODEzOTYxN30.1OekgKjJONMYsv0-k_tIkwlHsTczfNPhcT4u4b9sLEw')!,
      { global: { headers: { Authorization: authHeader } } }
    )
    const { data: { user: caller } } = await callerClient.auth.getUser()
    if (!caller) throw new Error('Not authenticated')

    const adminClient = createClient(
      Deno.env.get('adtgtzblbneqedkysurk.supabase.co')!,
      Deno.env.get('eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6ImFkdGd0emJsYm5lcWVka3lzdXJrIiwicm9sZSI6InNlcnZpY2Vfcm9sZSIsImlhdCI6MTc4MjU2MzYxNywiZXhwIjoyMDk4MTM5NjE3fQ.vcujznGbDP-klkHJ4ENksOGhZxwh1h-0nR87seVmaP0')!
    )

    const { data: callerProfile } = await adminClient
      .from('profiles').select('role').eq('id', caller.id).single()
    if (callerProfile?.role !== 'admin') throw new Error('Admin access required')

    // Create the new user
    const { email, password, full_name, role } = await req.json()
    if (!email || !password || !full_name || !role) throw new Error('Missing required fields')

    const { data: newUser, error: createErr } = await adminClient.auth.admin.createUser({
      email,
      password,
      email_confirm: true,
      user_metadata: { full_name, role }
    })
    if (createErr) throw createErr

    // Set profile
    await adminClient.from('profiles').upsert({
      id: newUser.user.id,
      email,
      full_name,
      role
    })

    return new Response(JSON.stringify({ success: true, id: newUser.user.id }), {
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    })

  } catch (err) {
    return new Response(JSON.stringify({ error: err.message }), {
      status: 400,
      headers: { ...corsHeaders, 'Content-Type': 'application/json' }
    })
  }
})
