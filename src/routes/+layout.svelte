<script>
    import Header from "$lib/Header.svelte";
    import Footer from "$lib/Footer.svelte";

    import { onMount, onDestroy } from "svelte";
    import { auth, db } from "../firebase";
    import { getDoc, doc, setDoc } from "@firebase/firestore";
    import { authStore } from "../store/store";

    const nonAuthRoutes = ["/", "product"];

    onMount(() => {
        const unsubscribe = auth.onAuthStateChanged(async (user) => {
            const currentPath = window.location.pathname;

            if (!user && !nonAuthRoutes.includes(currentPath)) {
                window.location.href = "/";
                return;
            }

            if (user && currentPath === "/") {
                window.location.href = "/dashboard";
                return;
            }

            if (!user) {
                return;
            }

            let dataToSetToStore;

            const docRef = doc(db, "users", user.uid);
            const docSnap = await getDoc(docRef);
            if (!docSnap.exists()) {
                const userRef = doc(db, "users", user.uid);
                dataToSetToStore = {
                    email: user.email,
                    pseudo: "",
                    todos: [],
                };
                await setDoc(userRef, dataToSetToStore, { merge: true });
            } else {
                const userData = docSnap.data();
                dataToSetToStore = userData;
            }

            authStore.update((curr) => {
                return {
                    ...curr,
                    user,
                    data: dataToSetToStore,
                    loading: false,
                };
            });
        })
        return unsubscribe;
    })
</script>
<div class="mainContainer">
<Header />
<slot></slot>
<Footer />
</div>
<style>
</style>