<script>
    import Navbar from '../components/Navbar.svelte';
    import DataLoader from '../components/DataLoader.svelte';
    import { writable } from 'svelte/store';
    
    let { children } = $props();
</script>

<style>
    /* Main Container */
    .main-page {
        display: flex;
        flex-direction: column;
        gap: 1rem;
        max-width: 1410px;
        margin: auto;
        /* padding: 1rem; */
    }
    /* Footer */
    footer {
        text-align: center;
        padding: 1rem;
        background: #333;
        color: white;
        margin-top: 1rem;
        border-radius: 8px;
    }
</style>

<div class="main-page">
    <!-- Navbar -->
    <Navbar title="Superstore Sales Dashboard" />

    <!-- Main Content -->
    {@render children()}
    
    <!-- Footer -->
    <footer>
        <p>© 2024 Superstore Sales Dashboard</p>
    </footer>
</div>
