FROM node:20-alpine AS builder 

WORKDIR /app

ARG VITE_SUPABASE_URL=https://dummy.supabase.co
ARG VITE_SUPABASE_ANON_KEY=dummy-key
ARG VITE_SUPABASE_SERVICE_ROLE_KEY=dummy-service-key

ENV VITE_SUPABASE_URL=$VITE_SUPABASE_URL
ENV VITE_SUPABASE_ANON_KEY=$VITE_SUPABASE_ANON_KEY
ENV VITE_SUPABASE_SERVICE_ROLE_KEY=$VITE_SUPABASE_SERVICE_ROLE_KEY

COPY package*.json ./
RUN npm install
COPY . .
RUN npm run build

FROM nginx:alpine
COPY --from=builder /app/dist /usr/share/nginx/html

EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]