CREATE TABLE standard_important_problems (
    id SERIAL PRIMARY KEY,
    problem_id INTEGER NOT NULL REFERENCES problems(id) ON DELETE CASCADE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    UNIQUE (problem_id)
);



CREATE TABLE user_problem_recommendations (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    problem_id INTEGER NOT NULL REFERENCES problems(id) ON DELETE CASCADE,

    reason TEXT,
    score NUMERIC(6, 3) DEFAULT 0,

    status TEXT DEFAULT 'pending',

    recommended_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    clicked_at TIMESTAMP,
    solved_at TIMESTAMP,
    dismissed_at TIMESTAMP,
    expires_at TIMESTAMP,

    UNIQUE (user_id, problem_id)
);



CREATE TABLE recommendation_events (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    recommendation_id INTEGER REFERENCES user_problem_recommendations(id) ON DELETE CASCADE,
    problem_id INTEGER REFERENCES problems(id) ON DELETE CASCADE,

    event_type TEXT NOT NULL,
    metadata JSONB,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);



CREATE TABLE user_notifications (
    id SERIAL PRIMARY KEY,

    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,

    notification_type TEXT NOT NULL,
    title TEXT NOT NULL,
    message TEXT NOT NULL,

    status TEXT DEFAULT 'unread',

    channel TEXT DEFAULT 'in_app',

    metadata JSONB DEFAULT '{}'::jsonb,

    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    sent_at TIMESTAMP,
    read_at TIMESTAMP,
    clicked_at TIMESTAMP,
    dismissed_at TIMESTAMP,
    expires_at TIMESTAMP
);