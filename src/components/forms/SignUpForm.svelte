<script>
	import Input from "@/components/forms/Input.svelte"

	import { ROLE_STATUS, appStatus } from "@/store.ts"

	const errors = {
		userName: "",
		email: "",
		password: "",
		confirmPassword: "",
	}

	async function handleUserSignUp(e) {
		const { userName, email, password, confirmPassword, role } = e.target.elements

		const data = {
			userName: userName.value.trim(),
			email: email.value.trim(),
			password: password.value.trim(),
			confirmPasswd: confirmPassword.value.trim(),
			role: role.value.trim(),
		}

		if (data.userName === "") {
			errors.userName = "El nombre es requerido"
			return
		} else errors.userName = ""

		if (data.userName.length < 3) {
			errors.userName = "El nombre debe ser de al menos 3 caracteres"
			return
		}

		if (data.email === "") {
			errors.email = "Este email es requerido"
			return
		} else errors.email = ""

		const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
		if (!emailRegex.test(data.email)) {
			errors.email = "El correo no es válido"
			return
		}

		if (data.password === "") {
			errors.password = "La contraseña es requerida"
			return
		} else errors.password = ""

		if (data.password.length < 8) {
			errors.password = "La contraseña debe ser de al menos 8 caracteres"
			return
		}

		if (data.password !== data.confirmPasswd) {
			errors.confirmPassword = "Las contraseñas no coinciden"
			return
		} else errors.confirmPassword = ""

		if (typeof window !== "undefined" && typeof localStorage !== "undefined") {
			const localRole = localStorage.getItem("role")
			if (localRole !== data.role) {
				console.log(
					"¡Atención! 🛑 ¡Este sistema está protegido por una muralla impenetrable! Los hackers encontrarán solo puertas cerradas. 🚪🔒 #SeguridadTotal"
				)
				return
			}
		}

		const formData = new FormData()
		formData.append("userName", data.userName)
		formData.append("email", data.email)
		formData.append("password", data.password)
		formData.append("role", data.role)

		const res = await fetch("/api/signup", {
			method: "POST",
			body: formData,
		})

		if (!res.ok) {
			const error = await res.json()
			errors.server = error.message
			return
		}

		const result = await res.json()

		if (result?.error) {
			errors.server = result.error
			return
		}

		window.location.href = "/login"
	}
</script>

<form class="flex flex-col items-center gap-y-12 lg:w-1/2" on:submit={(e) => handleUserSignUp(e)}>
	<div class="flex flex-col gap-y-2">
		<h1 class="flex justify-center font-sans text-2xl font-bold">
			Registro {$appStatus === ROLE_STATUS.STUDENT ? "alumno" : "profesor"}
		</h1>
		<p class="text-lm flex justify-center font-sans font-bold">
			¿Ya tienes cuenta?
			<a href="/login" class="ml-1 underline text-orange-500">Inicia sesión</a>
		</p>
	</div>
	{#if errors.server}
		<div class="mb-4 rounded-lg bg-yellow-50 p-4 text-sm text-yellow-800" role="alert">
			<span class="font-medium">{errors.server}</span>
		</div>
	{/if}
	<div class="relative flex justify-center w-80">
		<Input type="text" id="userName" content="Nombre completo:" />
	</div>
	{#if errors.userName}
		<div class="p-4 mb-4 w-80 text-sm text-red-800 rounded-lg bg-red-50" role="alert">
			<span class="font-medium">{errors.userName}</span>
		</div>
	{/if}
	<div class="relative flex justify-center w-80">
		<Input type="email" id="email" content="Correo:" />
	</div>
	{#if errors.email}
		<div class="p-4 mb-4 w-80 text-sm text-red-800 rounded-lg bg-red-50" role="alert">
			<span class="font-medium">{errors.email}</span>
		</div>
	{/if}
	<div class="relative flex justify-center w-80">
		<Input type="password" id="password" content="Contraseña:" />
	</div>
	{#if errors.password}
		<div class="p-4 mb-4 w-80 text-sm text-red-800 rounded-lg bg-red-50" role="alert">
			<span class="font-medium">{errors.password}</span>
		</div>
	{/if}
	<div class="relative flex justify-center w-80">
		<Input type="password" id="confirmPassword" content="Confirmar contraseña:" />
	</div>
	{#if errors.confirmPassword}
		<div class="p-4 mb-4 w-80 text-sm text-red-800 rounded-lg bg-red-50" role="alert">
			<span class="font-medium">{errors.confirmPassword}</span>
		</div>
	{/if}
	<Input type="hidden" id="role" value={`${$appStatus}`} />
	<input
		type="submit"
		value="Registrarse"
		class="mx-auto w-56 justify-center rounded-full bg-orange-500 px-4 py-2 text-lg font-bold text-white hover:bg-orange-600 cursor-pointer"
	/>
</form>
