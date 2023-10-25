<script lang="ts">
    import { onMount } from "svelte";
    import { writable } from "svelte/store";

    type Post = {
        title: string;
        text: string;
        created: string;
        edited: string;
        id: string;
    };

    // Сохраняем посты в хранилище
    const posts = writable<Post[]>([]);

    // Хранит выбранный пост
    const selectedPost = writable<Post | null>(null);

    //Функция для генерации случайного id для поста
    function generateRandomId() {
        const characters =
            "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789";
        let id = "";
        for (let i = 0; i < 8; i++) {
            const randomIndex = Math.floor(Math.random() * 8);
            id += characters.charAt(randomIndex);
        }
        return id;
    }

    //Функция добавления поста
    function savePost(title: string, text: string) {
        const now = new Date().toLocaleString();
        const id = generateRandomId();
        const post = {
            title,
            text,
            created: now,
            edited: now,
            id: id,
        };

        posts.update((prevPosts) => {
            const newPosts = [...prevPosts, post];
            localStorage.setItem("posts", JSON.stringify(newPosts));
            return newPosts;
        });
    }

    //Функция редактирования поста
    function updatePost(title: string, text: string) {
        selectedPost.update((post) => {
            if (post) {
                const updatedPost = {
                    ...post,
                    title,
                    text,
                    edited: new Date().toLocaleString(),
                };
                return updatedPost;
            }
            return null;
        });

        posts.update((prevPosts) => {
            const updatedPosts = prevPosts.map((post) => {
                if (post.id === $selectedPost?.id) {
                    return {
                        ...post,
                        title,
                        text,
                        edited: new Date().toLocaleString(),
                    };
                }
                return post;
            });

            localStorage.setItem("posts", JSON.stringify(updatedPosts));
            return updatedPosts;
        });
    }

    //Функция удаления поста
    function deletePost(id: string) {
        posts.update((prevPosts) => {
            const updatedPosts = prevPosts.filter((p) => p.id !== id);
            localStorage.setItem("posts", JSON.stringify(updatedPosts));
            return updatedPosts;
        });

        selectedPost.update((selected) => {
            if (selected?.id === id) {
                return null;
            }
            return selected;
        });
    }

    // Загрузка сохраненных постов
    onMount(() => {
        const savedPosts = localStorage.getItem("posts");
        if (savedPosts) {
            const parsedPosts = JSON.parse(savedPosts);
            posts.set(parsedPosts);
        }
    });

    let titleInput: HTMLInputElement;
    let textInput: HTMLTextAreaElement;

    // Обработчик клика на посте
    function handlePostClick(post: {
        title: string;
        text: string;
        created: string;
        edited: string;
        id: string;
    }) {
        selectedPost.set(post);
        setTimeout(() => {
            titleInput.value = post.title;
            textInput.value = post.text;
        }, 0);
    }

    //Функция клика на пост для режима доступности
    function handlePostKeyDown(
        post: {
            title: string;
            text: string;
            created: string;
            edited: string;
            id: string;
        },
        event: KeyboardEvent
    ) {
        if (event.key === "Enter" || event.key === " ") {
            handlePostClick(post);
        }
    }

    //Функция для кнопки сохранения поста или реадктирования
    function handleSubmit() {
        const title = titleInput.value;
        const text = textInput.value;

        if ($selectedPost) {
            updatePost(title, text);
        } else {
            savePost(title, text);
            titleInput.value = "";
            textInput.value = "";
        }
    }
</script>

<main>
    <h1>Блог с постами</h1>
    <h2>Общий список постов</h2>
    {#each $posts as post (post.id)}
        <div
            class="postCard postCardHover"
            role="button"
            tabindex="0"
            on:click={() => handlePostClick(post)}
            on:keydown={(event) => handlePostKeyDown(post, event)}
            aria-label="Выбрать пост"
        >
            <h3>{post.title}</h3>
            <p>Создано: {post.created}</p>
            <p>Отредактировано: {post.edited}</p>
            <button on:click={() => deletePost(post.id)}>УДАЛИТЬ</button>
        </div>
    {/each}

    {#if $selectedPost}
        <h2>Выбранный пост:</h2>
        <div class="postCard">
            <h3>{$selectedPost.title}</h3>
            <p>Текст: {$selectedPost.text}</p>
            <p>Создано: {$selectedPost.created}</p>
            <p>Отредактировано: {$selectedPost.edited}</p>
        </div>

        <h2>Редактирование поста</h2>
        <form on:submit|preventDefault={handleSubmit}>
            <label for="edit-title">Заголовок:</label>
            <input
                type="text"
                id="edit-title"
                required
                bind:this={titleInput}
            />

            <label for="edit-text">Текст:</label>
            <textarea id="edit-text" rows="4" required bind:this={textInput} />

            <div>
                <button type="submit">Сохранить</button>
                <button type="button" on:click={() => selectedPost.set(null)}>
                    Вернуться к созданию поста
                </button>
            </div>
        </form>
    {:else}
        <h2>Создание нового поста</h2>
        <form on:submit|preventDefault={handleSubmit}>
            <label for="title">Заголовок:</label>
            <input type="text" id="title" required bind:this={titleInput} />

            <label for="text">Текст:</label>
            <textarea id="text" rows="4" required bind:this={textInput} />

            <button type="submit">Сохранить</button>
        </form>
    {/if}
</main>

<style lang="scss">
    $primary-color: #2196f3;
    @mixin button {
        button {
            padding: 8px;
            border: none;
            border-radius: 4px;
            background-color: $primary-color;
            color: #fff;
            font-size: 14px;
            cursor: pointer;
        }

        button:hover {
            box-shadow: 4px 2px 2px rgb(145, 145, 145);
        }
    }

    main {
        display: flex;
        flex-direction: column;
        align-items: center;
        justify-content: center;

        h1 {
            font-size: 24px;
            margin-bottom: 16px;
            color: $primary-color;
        }

        h2 {
            font-size: 20px;
            margin-top: 32px;
            margin-bottom: 16px;
            color: $primary-color;
        }

        .postCard {
            width: 330px;
            cursor: pointer;
            padding: 16px;
            border-radius: 4px;
            background-color: #f5f5f5;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            margin-bottom: 8px;
            box-sizing: border-box;

            h3 {
                font-size: 16px;
                margin-bottom: 8px;
            }

            p {
                font-size: 14px;
                color: #777;
                margin-bottom: 4px;
            }

            @include button;
        }

        .postCardHover:hover {
            box-shadow: 10px 5px 5px lightgrey;
        }

        form {
            display: flex;
            flex-direction: column;
            align-items: flex-start;
            width: 330px;
            margin-top: 16px;

            label {
                font-size: 14px;
                margin-bottom: 4px;
            }

            input,
            textarea {
                width: 100%;
                padding: 8px;
                border: 1px solid #ccc;
                border-radius: 4px;
                font-size: 14px;
                margin-bottom: 8px;
                box-sizing: border-box;
            }

            div {
                display: flex;
                width: 100%;
                justify-content: space-between;
                padding: 0;
            }

            @include button;
        }
    }
</style>
